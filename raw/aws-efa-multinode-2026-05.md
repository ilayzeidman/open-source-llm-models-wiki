# AWS EFA + multi-node networking — May 2026

## Authoritative URLs

- EFA overview: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/efa.html
- Bandwidth per accelerated instance:
  https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/efa-acc-inst-types.html
- EFA + NCCL setup:
  https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/efa-start-nccl.md
- EKS + EFA: https://docs.aws.amazon.com/eks/latest/userguide/node-efa.html
- aws-ofi-nccl: https://github.com/aws/aws-ofi-nccl
- p5en launch announcement (EFAv3):
  https://aws.amazon.com/blogs/aws/new-amazon-ec2-p5en-instances-with-nvidia-h200-tensor-core-gpus-and-efav3-networking/
- AWS EC2 G5: https://aws.amazon.com/ec2/instance-types/g5/
- AWS EC2 G6e: https://aws.amazon.com/ec2/instance-types/g6e/
- NCCL: https://github.com/NVIDIA/nccl

## What EFA is

"An Elastic Fabric Adapter (EFA) is a network device... [that] enables you to
achieve the application performance of an on-premises AI/ML or HPC cluster,
with the scalability, flexibility, and elasticity provided by the AWS Cloud."

EFA bypasses the OS via **Libfabric** and uses the **SRD** (Scalable Reliable
Datagram) protocol — connectionless, reliable, lossless. Supports
**NCCL ≥ 2.4.2**, **NIXL ≥ 1.0.0**, Open MPI/Intel MPI.

Constraint: "**EFA traffic can't cross Availability Zones or VPCs.**" For
multi-node training/inference, all instances must be in the same AZ (and
ideally a **cluster placement group**).

## EFA vs ENA

| | ENA | EFA (with ENA) | EFA-only |
|---|---|---|---|
| IP networking | Yes | Yes | No |
| OS-bypass / SRD | No | Yes | Yes |
| Primary NIC OK | Yes | Yes | No |
| Use as primary | always | yes | no |

## Bandwidth per instance type (2026)

| Instance | GPUs | Per-GPU mem | EFA bandwidth (aggregate) | NVLink | EFA gen |
|---|---|---|---|---|---|
| `g5.xlarge`–`g5.16xlarge` | 1× A10G | 24 GB | **No EFA** (Up to 10–25 Gbps ENA) | No | n/a |
| `g5.12xlarge` | 4× A10G | 24 GB | **No EFA** (40 Gbps ENA) | No | n/a |
| `g5.24xlarge` | 4× A10G | 24 GB | **No EFA** (50 Gbps ENA) | No | n/a |
| `g5.48xlarge` | 8× A10G | 24 GB | **No EFA** (100 Gbps ENA) | No | n/a |
| `g6.8xlarge`–`g6.48xlarge` | 1–8× L4 | 24 GB | **EFA supported** | No | EFA |
| `g6e.8xlarge`–`g6e.48xlarge` | 1–8× L40S | 48 GB | **EFA**, up to 400 Gbps on g6e.48xlarge | No | EFA |
| `p4d.24xlarge` | 8× A100-40GB | 40 GB | 400 Gbps (4× 100) | NVSwitch | EFAv1 |
| `p4de.24xlarge` | 8× A100-80GB | 80 GB | 400 Gbps | NVSwitch | EFAv1 |
| `p5.48xlarge` | 8× H100 | 80 GB | **3,200 Gbps** (32 NICs × 100 Gbps); up to 800 Gbps for IP | NVSwitch | EFAv2 |
| `p5e.48xlarge` | 8× H200 | 141 GB | **3,200 Gbps** | NVSwitch | EFAv2 |
| `p5en.48xlarge` | 8× H200 | 141 GB | **3,200 Gbps** with EFAv3 + Nitro v5; **up to 35% latency improvement vs P5**; PCIe Gen5; 4× CPU↔GPU bandwidth | NVSwitch (900 GB/s) | EFAv3 |
| `p6-b200.48xlarge` | 8× B200 | 192 GB | 3,200 Gbps EFA + up to 1,600 Gbps ENA (8 NICs × 400 Gbps EFA each) | NVLink | EFA |
| `p6-b300.48xlarge` | 8× B300 | — | **6,400 Gbps EFA**, 3,870 Gbps ENA (17 NICs) | NVLink | EFA |
| `p6e-gb200.36xlarge` | 4× GB200 | — | NCI 1,3,5,7,9,11,13,15 → 400 Gbps EFA each | NVLink | EFA |

### Critical wiki callout
**g5 (the project's primary deployment target) does NOT support EFA.**
`g5.xlarge` is single-A10G; for multi-node serving on g5, you only have ENA
(≤25 Gbps on small sizes, 100 Gbps on g5.48xlarge), which is fine for **PP**
between nodes or **independent DP replicas**, but the wrong choice for **TP
across nodes**. The official guidance ("for nodes without NVLink, leverage
pipeline parallelism") applies directly.

## Cluster placement groups

For multi-node ML/inference: launch all instances into a **cluster placement
group** to get same-rack low-latency. AWS docs: "we do recommend running your
EFA-enabled instances in a cluster placement group as it launches the
instances into a low-latency group in a single Availability Zone." Combine
with **Capacity Reservations** for guaranteed scaling.

## EFA on EKS

- `eksctl` flag: `efaEnabled: true` on a managed nodegroup → auto-installs
  the AWS EFA Kubernetes Device Plugin (DaemonSet `aws-efa-k8s-device-plugin-
  daemonset`).
- Pod requests resource: `vpc.amazonaws.com/efa: N` (N = EFA NICs available
  on instance) plus `hugepages-2Mi: 5120Mi`. P5/P5e instances allocate
  5128 × 2 MiB Huge Pages.
- VPC CNI ≥ 1.18.5 required for EFA-only; ≥ 1.7.10 for multi-EFA instances.
- For p6-b200, EFA Device Plugin ≥ v0.5.6.
- Recommended pattern: Kubeflow MPI Operator + `nccl-tests` to validate.

## NCCL over EFA — the AWS plugin

**aws-ofi-nccl** (https://github.com/aws/aws-ofi-nccl) is the libfabric-NCCL
bridge. Without it, NCCL falls back to TCP and you lose all the EFA benefit.

"AWS OFI NCCL is a plug-in which enables EC2 developers to use libfabric as
a network provider while running NVIDIA's NCCL based applications."

- Installs to `/opt/amazon/ofi-nccl/` via `aws-efa-installer`.
- Requires Libfabric `FI_EP_RDM` + `FI_TAGGED` + `FI_HMEM` for GPU paths.
- Confirm activation in NCCL logs: `NCCL INFO NET/OFI Selected Provider is
  efa*`.
- NCCL tuner plugin: `NCCL_TUNER_PLUGIN=/opt/amazon/ofi-nccl/lib/x86_64-linux-gnu/libnccl-ofi-tuner.so`.
- p4d/p5 path: `FI_EFA_USE_DEVICE_RDMA=1` to use device RDMA for one/two-sided
  transfer.

## Critical NCCL env vars

| Variable | Purpose |
|---|---|
| `NCCL_DEBUG=INFO\|TRACE` | Enable diagnostics; verify EFA selection |
| `NCCL_SOCKET_IFNAME` | Pin control plane to specific NIC (e.g. `^lo,docker0`) |
| `NCCL_IB_HCA` | Restrict to specific InfiniBand HCAs (off-AWS) |
| `NCCL_NET_PLUGIN` | Override net plugin; AWS uses ofi |
| `NCCL_BUFFSIZE=8388608` | Tuning for large messages on EFA |
| `NCCL_P2P_NET_CHUNKSIZE=524288` | Chunk size, EFA-tuned |
| `FI_EFA_USE_DEVICE_RDMA=1` | Required on p4d/p5 |
| `FI_PROVIDER=efa` | Force EFA |
| `GLOO_SOCKET_IFNAME=eth0` | (vLLM EP docs note for IB clusters) |

## Specific impact for the wiki's primary target

g5.xlarge → multi-node serving: do NOT plan TP across g5 instances. The lack
of EFA + lack of NVLink means TP all-reduce traffic goes over ENA at
≤25 Gbps, which is catastrophic for any nontrivial model.

Multi-instance scaling on g5 has three valid shapes:
1. **Multiple independent replicas** — one model per g5, behind a router
   (vllm-router / production-stack / AIBrix). Best fit for the "scale 1 → 2 →
   5 machines" question.
2. **PP across g5 instances** — works for very large models that don't fit
   on a single g5 even at AWQ INT4. The lower comm volume of PP makes ENA
   tolerable, though latency suffers.
3. **DP for replicas + EP for MoE** — only relevant for MoE models on a
   different instance class.

For TP-multi-node, use **g6e** (EFA) or **p4d/p5/p6** (EFA + NVLink/NVSwitch).
