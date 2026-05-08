---
tags: [hardware, aws, networking, multi-node]
last_updated: 2026-05-08
source_count: 1
---

# AWS Elastic Fabric Adapter (EFA)

The AWS network device that enables multi-node training and inference at
near-on-prem-cluster latency. **The single most important fact for this
wiki**: g5 (the primary baseline) does **not** support EFA, while g6e / p4d /
p5 / p5e / p5en / p6 do. This page is the canonical reference for which
AWS GPU instances support multi-node TP and which don't.
[Source: [[sources/aws-efa-multinode-2026-05]]]

## What EFA is

> "An Elastic Fabric Adapter (EFA) is a network device... [that] enables you
> to achieve the application performance of an on-premises AI/ML or HPC
> cluster, with the scalability, flexibility, and elasticity provided by the
> AWS Cloud."

EFA bypasses the OS via **Libfabric** and uses the **SRD** (Scalable Reliable
Datagram) protocol — connectionless, reliable, lossless. Supports
**NCCL ≥ 2.4.2**, **NIXL ≥ 1.0.0**, Open MPI/Intel MPI.

Hard constraint: "**EFA traffic can't cross Availability Zones or VPCs.**"
For multi-node training/inference, all instances must be in the same AZ —
and ideally a **cluster placement group**.

## EFA vs ENA

| | ENA | EFA (with ENA) | EFA-only |
|---|---|---|---|
| IP networking | Yes | Yes | No |
| OS-bypass / SRD | No | Yes | Yes |
| Primary NIC OK | Yes | Yes | No |
| Use as primary | always | yes | no |

## Bandwidth per instance type (May 2026)

| Instance | GPUs | Per-GPU mem | EFA bandwidth (aggregate) | NVLink | EFA gen |
|---|---|---|---|---|---|
| g5.xlarge–g5.16xlarge | 1× A10G | 24 GB | **No EFA** (10–25 Gbps ENA) | No | n/a |
| g5.12xlarge | 4× A10G | 24 GB | **No EFA** (40 Gbps ENA) | No | n/a |
| g5.24xlarge | 4× A10G | 24 GB | **No EFA** (50 Gbps ENA) | No | n/a |
| g5.48xlarge | 8× A10G | 24 GB | **No EFA** (100 Gbps ENA) | No | n/a |
| g6.8xlarge–g6.48xlarge | 1–8× L4 | 24 GB | **EFA supported** | No | EFA |
| g6e.8xlarge–g6e.48xlarge | 1–8× L40S | 48 GB | **EFA**, up to 400 Gbps on g6e.48xlarge | No | EFA |
| p4d.24xlarge | 8× A100-40G | 40 GB | 400 Gbps | NVSwitch | EFAv1 |
| p4de.24xlarge | 8× A100-80G | 80 GB | 400 Gbps | NVSwitch | EFAv1 |
| p5.48xlarge | 8× H100 | 80 GB | **3,200 Gbps** | NVSwitch | EFAv2 |
| p5e.48xlarge | 8× H200 | 141 GB | **3,200 Gbps** | NVSwitch | EFAv2 |
| p5en.48xlarge | 8× H200 | 141 GB | **3,200 Gbps** + EFAv3; "up to 35% latency improvement vs P5"; PCIe Gen5 | NVSwitch (900 GB/s) | EFAv3 |
| p6-b200.48xlarge | 8× B200 | 192 GB | 3,200 Gbps EFA + 1,600 Gbps ENA | NVLink | EFA |
| p6-b300.48xlarge | 8× B300 | — | **6,400 Gbps EFA**, 3,870 Gbps ENA | NVLink | EFA |
| p6e-gb200.36xlarge | 4× GB200 | — | 400 Gbps EFA per NCI (some pairs) | NVLink | EFA |

## **Critical**: g5 does not support EFA

g5.xlarge is single-A10G. For multi-node serving on g5, only ENA is
available (≤25 Gbps small sizes, 100 Gbps on g5.48xlarge). This is fine for:

- **PP** between nodes (PP transfer volume is `B·S·H·(P-1)·bytes`).
- **Independent DP replicas** (no inter-node traffic).

This is **wrong** for:

- **TP across nodes** (TP per-layer AllReduce volume saturates ENA quickly).

The official vLLM guidance applies directly: "for nodes without NVLink,
leverage pipeline parallelism." On g5, you have neither EFA nor NVLink at
the instance boundary. Don't run TP across two g5 instances.

For real multi-node TP-capable AWS deployments, step up to:
- **g6e** for L40S 48 GB single-GPU and EFA across multi-GPU instances.
- **p4d / p4de** for A100 NVSwitch + 400 Gbps EFA.
- **p5 / p5e / p5en** for H100/H200 NVSwitch + 3,200 Gbps EFA.
- **p6** for B200/B300 NVLink-5 + 3,200–6,400 Gbps EFA.

## NCCL over EFA — the AWS plugin

aws-ofi-nccl: https://github.com/aws/aws-ofi-nccl is the libfabric–NCCL
bridge. Without it, NCCL falls back to TCP and you lose all the EFA
benefit.

> "AWS OFI NCCL is a plug-in which enables EC2 developers to use libfabric
> as a network provider while running NVIDIA's NCCL based applications."

Setup notes:
- Installs to `/opt/amazon/ofi-nccl/` via `aws-efa-installer`.
- Confirm activation in NCCL logs: `NCCL INFO NET/OFI Selected Provider is
  efa*`.
- p4d/p5: `FI_EFA_USE_DEVICE_RDMA=1` to use device RDMA.
- NCCL tuner plugin (recommended):
  `NCCL_TUNER_PLUGIN=/opt/amazon/ofi-nccl/lib/x86_64-linux-gnu/libnccl-ofi-tuner.so`.

## Cluster placement groups

For multi-node ML/inference: launch all instances into a **cluster placement
group** for same-rack low latency. AWS docs:

> "we do recommend running your EFA-enabled instances in a cluster placement
> group as it launches the instances into a low-latency group in a single
> Availability Zone."

Combine with **Capacity Reservations** for guaranteed scaling.

## EFA on EKS

- `eksctl` flag: `efaEnabled: true` on a managed nodegroup auto-installs
  the AWS EFA Kubernetes Device Plugin (DaemonSet
  `aws-efa-k8s-device-plugin-daemonset`).
- Pod resources:
  ```yaml
  resources:
    limits:
      vpc.amazonaws.com/efa: 32        # for p5.48xlarge (32 NICs)
      hugepages-2Mi: 5120Mi             # 5128 × 2 MiB on P5/P5e
  ```
- VPC CNI ≥ 1.18.5 required for EFA-only; ≥ 1.7.10 for multi-EFA.
- For p6-b200, EFA Device Plugin ≥ v0.5.6.
- Validate with Kubeflow MPI Operator + `nccl-tests`.

## Critical NCCL env vars

| Variable | Purpose |
|---|---|
| `NCCL_DEBUG=INFO\|TRACE` | Enable diagnostics; verify EFA selection |
| `NCCL_SOCKET_IFNAME` | Pin control plane to specific NIC |
| `NCCL_NET_PLUGIN` | Override net plugin |
| `NCCL_BUFFSIZE=8388608` | EFA tuning |
| `NCCL_P2P_NET_CHUNKSIZE=524288` | EFA chunk size |
| `FI_EFA_USE_DEVICE_RDMA=1` | Required on p4d/p5 |
| `FI_PROVIDER=efa` | Force EFA |

## Sanity check after launch

```bash
# Confirm EFA device present
fi_info -p efa

# Check NCCL is using EFA (verbose)
NCCL_DEBUG=INFO ... your-vllm-or-test-command 2>&1 | grep "NET/OFI"
# Expect: "NET/OFI Selected Provider is efa"

# nccl-tests across nodes
mpirun -np 16 -hostfile ./hosts \
  -x NCCL_DEBUG=INFO -x FI_PROVIDER=efa \
  /opt/nccl-tests/build/all_reduce_perf -b 8 -e 8G -f 2 -g 1
```

## Related

- [[hardware/aws-gpu-landscape]] — full instance × price table
- [[hardware/multi-gpu-options]]
- [[concepts/parallelism-strategies]]
- [[infrastructure/leaderworkerset]]
- [[comparisons/scaling-1-to-5-machines]]

## Sources

- [[sources/aws-efa-multinode-2026-05]]
