---
tags: [source, hardware, aws, networking, multi-node]
source_path: raw/aws-efa-multinode-2026-05.md
source_url: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/efa.html
ingested: 2026-05-08
last_updated: 2026-05-08
---

# AWS EFA + multi-node networking (May 2026)

Authoritative bandwidth/EFA matrix for AWS NVIDIA-GPU instances, plus the
NCCL/aws-ofi-nccl plugin requirements for multi-node TP/EP serving.

## Key claims

### What EFA is

- "An Elastic Fabric Adapter (EFA) is a network device... [that] enables
  you to achieve the application performance of an on-premises AI/ML or HPC
  cluster."
- OS-bypass via **Libfabric**; uses **SRD** (Scalable Reliable Datagram) —
  connectionless, reliable, lossless.
- Supports **NCCL ≥ 2.4.2**, **NIXL ≥ 1.0.0**, Open MPI/Intel MPI.
- Hard constraint: "EFA traffic can't cross Availability Zones or VPCs."

### EFA support / bandwidth per instance

| Instance | GPUs | Per-GPU mem | EFA aggregate | NVLink | EFA gen |
|---|---|---|---|---|---|
| g5.xlarge–g5.16xlarge | 1× A10G | 24 GB | **No EFA** (10–25 Gbps ENA) | No | n/a |
| g5.12xlarge | 4× A10G | 24 GB | **No EFA** (40 Gbps ENA) | No | n/a |
| g5.24xlarge | 4× A10G | 24 GB | **No EFA** (50 Gbps ENA) | No | n/a |
| g5.48xlarge | 8× A10G | 24 GB | **No EFA** (100 Gbps ENA) | No | n/a |
| g6.8xlarge–g6.48xlarge | 1–8× L4 | 24 GB | EFA supported | No | EFA |
| g6e.8xlarge–g6e.48xlarge | 1–8× L40S | 48 GB | EFA, up to 400 Gbps on g6e.48xlarge | No | EFA |
| p4d.24xlarge | 8× A100-40G | 40 GB | 400 Gbps | NVSwitch | EFAv1 |
| p4de.24xlarge | 8× A100-80G | 80 GB | 400 Gbps | NVSwitch | EFAv1 |
| p5.48xlarge | 8× H100 | 80 GB | **3,200 Gbps** | NVSwitch | EFAv2 |
| p5e.48xlarge | 8× H200 | 141 GB | **3,200 Gbps** | NVSwitch | EFAv2 |
| p5en.48xlarge | 8× H200 | 141 GB | **3,200 Gbps** + EFAv3; "up to 35% latency improvement vs P5"; PCIe Gen5 | NVSwitch (900 GB/s) | EFAv3 |
| p6-b200.48xlarge | 8× B200 | 192 GB | 3,200 Gbps EFA + 1,600 Gbps ENA | NVLink | EFA |
| p6-b300.48xlarge | 8× B300 | — | **6,400 Gbps EFA**, 3,870 Gbps ENA | NVLink | EFA |
| p6e-gb200.36xlarge | 4× GB200 | — | NCI 1,3,5,7,9,11,13,15 → 400 Gbps EFA each | NVLink | EFA |

### Wiki callout (g5 = primary baseline)

**g5 does NOT support EFA.** g5.xlarge is single-A10G; for multi-node serving
on g5, only ENA is available (≤25 Gbps small sizes, 100 Gbps on g5.48xlarge).

This is fine for:
- **PP** between nodes (PP transfer volume is `B·S·H·(P-1)·bytes`).
- **Independent DP replicas** (no inter-node traffic).

This is wrong for:
- **TP across nodes** (TP per-layer AllReduce volume saturates ENA quickly).

The official guidance ("for nodes without NVLink, leverage pipeline
parallelism") applies directly.

### NCCL over EFA — aws-ofi-nccl plugin

- aws-ofi-nccl: https://github.com/aws/aws-ofi-nccl
- "AWS OFI NCCL is a plug-in which enables EC2 developers to use libfabric
  as a network provider while running NVIDIA's NCCL based applications."
- Without it, NCCL falls back to TCP and EFA benefit is lost.
- Installs to `/opt/amazon/ofi-nccl/` via `aws-efa-installer`.
- Confirm via `NCCL_DEBUG=TRACE` log: `NET/OFI Selected Provider is efa*`.
- p4d/p5: `FI_EFA_USE_DEVICE_RDMA=1` for device RDMA.
- NCCL tuner plugin: `NCCL_TUNER_PLUGIN=/opt/amazon/ofi-nccl/lib/...
  /libnccl-ofi-tuner.so`.

### Cluster placement groups

For multi-node ML/inference: launch into a **cluster placement group** for
same-rack low latency. AWS docs: "we do recommend running your EFA-enabled
instances in a cluster placement group as it launches the instances into a
low-latency group in a single Availability Zone." Combine with **Capacity
Reservations** for guaranteed scaling.

### EFA on EKS

- `eksctl efaEnabled: true` on a managed nodegroup auto-installs the AWS EFA
  K8s Device Plugin (DaemonSet `aws-efa-k8s-device-plugin-daemonset`).
- Pod resources: `vpc.amazonaws.com/efa: N` (N = NICs available) plus
  `hugepages-2Mi: 5120Mi` (P5/P5e: 5128 × 2 MiB).
- VPC CNI ≥ 1.18.5 for EFA-only; ≥ 1.7.10 for multi-EFA instances.
- Validate with Kubeflow MPI Operator + `nccl-tests`.

### Critical NCCL env vars

| Variable | Purpose |
|---|---|
| `NCCL_DEBUG=INFO\|TRACE` | Enable diagnostics; verify EFA selection |
| `NCCL_SOCKET_IFNAME` | Pin control plane to specific NIC |
| `NCCL_NET_PLUGIN` | Override net plugin |
| `NCCL_BUFFSIZE=8388608` | EFA tuning |
| `NCCL_P2P_NET_CHUNKSIZE=524288` | EFA chunk size |
| `FI_EFA_USE_DEVICE_RDMA=1` | Required on p4d/p5 |
| `FI_PROVIDER=efa` | Force EFA |

### Citation URLs

- https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/efa.html
- https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/efa-acc-inst-types.html
- https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/efa-start-nccl.md
- https://docs.aws.amazon.com/eks/latest/userguide/node-efa.html
- https://github.com/aws/aws-ofi-nccl
- https://aws.amazon.com/blogs/aws/new-amazon-ec2-p5en-instances-with-nvidia-h200-tensor-core-gpus-and-efav3-networking/
- https://github.com/NVIDIA/nccl

## Pages updated on ingest

- [[hardware/aws-efa]] (NEW)
- [[hardware/multi-gpu-options]]
- [[hardware/aws-gpu-landscape]]
- [[comparisons/scaling-1-to-5-machines]] (NEW)
