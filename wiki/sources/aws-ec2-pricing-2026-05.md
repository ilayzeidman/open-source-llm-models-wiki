---
tags: [source, hardware, aws, pricing]
source_path: raw/aws-g5-pricing-2026-05.md, raw/aws-g5-instance-types-2026-05.md, raw/aws-p4-instance-types.md, raw/aws-p5-instance-types.md
source_url: https://aws.amazon.com/ec2/instance-types/g5/, https://instances.vantage.sh/
ingested: 2026-05-07
last_updated: 2026-05-07
---

# AWS EC2 G5 / P4 / P5 pricing snapshot (2026-05)

Cross-checked us-east-1 on-demand pricing for the GPU instances relevant to a single-A10G research wiki. AWS's own page lists specs but not all on-demand prices; instances.vantage.sh mirrors the pricing API and was cross-checked.

## Key claims

### G5 family (us-east-1, on-demand, Linux)

| Instance | GPUs | Total VRAM | $/hr | $/GPU-hr |
|---|---|---:|---:|---:|
| g5.xlarge | 1× A10G | 24 GB | $1.006 | $1.006 |
| g5.2xlarge | 1× A10G | 24 GB | $1.212 | $1.212 |
| g5.4xlarge | 1× A10G | 24 GB | $1.624 | $1.624 |
| g5.8xlarge | 1× A10G | 24 GB | $2.448 | $2.448 |
| g5.16xlarge | 1× A10G | 24 GB | $4.096 | $4.096 |
| g5.12xlarge | 4× A10G | 96 GB | $5.672 | $1.418 |
| g5.24xlarge | 4× A10G | 96 GB | $8.144 | $2.036 |
| g5.48xlarge | 8× A10G | 192 GB | $16.288 | $2.036 |

**g5.xlarge is the cheapest single-A10G option.** Stepping up within the 1× A10G tier (g5.2xlarge through g5.16xlarge) buys more vCPU/RAM/NVMe/network — **not** more VRAM.

### P4 / P5 family

| Instance | GPUs | Total VRAM | $/hr (list, on-demand) |
|---|---|---:|---:|
| p4d.24xlarge | 8× A100 40 GB | 320 GB | $32.7726 |
| p4de.24xlarge | 8× A100 80 GB | 640 GB | $40.9657 |
| p5.48xlarge | 8× H100 80 GB | 640 GB | $98.32 |
| p5e.48xlarge | 8× H200 141 GB | 1,128 GB | Capacity-Blocks-only ($39.799/hr reservation fee) |
| p5en.48xlarge | 8× H200 141 GB | 1,128 GB | Capacity-Blocks-only ($45.768/hr reservation fee) |

Vantage.sh's headline P4/P5 numbers were lower (~$22/hr p4d, ~$55/hr p5.48xl) but those appear to be Savings-Plan / Reserved rates, not list on-demand. AWS's own pricing page gives the list prices above. P5e/P5en are Capacity-Blocks-only.

### G5 multi-GPU caveat: no NVLink

g5.12xlarge / g5.24xlarge / g5.48xlarge connect their A10Gs over **PCIe Gen4 only** — no NVLink, no NVSwitch. Tensor parallelism works but communication is much slower than A100/H100 boxes, which has real implications for sub-second latency at TP > 1.

### Spot pricing

Typical G5 spot discounts: 30–70% off on-demand. Pull current rates from AWS Spot Advisor or the AWS pricing API for production planning; spot rates fluctuate by region and time.

## Pages updated on ingest
- [[hardware/a10g-g5xlarge]]
- [[hardware/multi-gpu-options]]
- [[wiki/comparisons/tool-calling-models-on-a10g]]
- [[wiki/overview]]
