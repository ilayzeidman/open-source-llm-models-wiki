---
tags: [source, aws, hardware, gpu]
source_path: raw/aws-g4dn-instance-types.md, raw/aws-g6-instance-types.md, raw/aws-g6e-instance-types.md, raw/aws-p4-instance-types.md, raw/aws-p5-instance-types.md, raw/aws-p6-b200-instance.md
source_url: https://aws.amazon.com/ec2/pricing/on-demand/
ingested: 2026-05-07
last_updated: 2026-05-07
---

# AWS Extended GPU Pricing — G4dn/G5/G6/G6e/P4/P5/P6 (May 2026)

Combined ingest of every NVIDIA-GPU EC2 family that ships open-source LLMs at AWS.
Prices are us-east-1 on-demand list, May 2026. Spot is typically 30–70% off; AWS
Capacity Blocks for ML are required for some P5e/P5en/P6-B200 SKUs.

## Key claims

- **g4dn.xlarge** (1× T4 16 GB, sm_75) = **$0.526/hr** — cheapest AWS GPU box.
- **g5.xlarge** (1× A10G 24 GB, sm_86) = **$1.006/hr** — original baseline.
- **g6.xlarge** (1× L4 24 GB, sm_89) = **$0.8048/hr** — **cheaper than G5**, FP8 native,
  but lower memory bandwidth (300 vs 600 GB/s) — typically *slower* on decode-bound LLMs.
- **g6e.xlarge** (1× L40S **48 GB**, sm_89) = **$1.861/hr** — **cheapest 48 GB single-GPU
  on AWS**. Unlocks 24B–32B models at FP16/BF16 single-GPU and 70B AWQ INT4 single-node
  on g6e.48xlarge.
- **p4d.24xlarge** (8× A100 40 GB) = $32.7726/hr; **p4de.24xlarge** (8× A100 80 GB) = $40.9657/hr.
- **p5.48xlarge** (8× H100 80 GB) = $98.32/hr list — first AWS option with NVLink + native FP8.
- **p5e.48xlarge / p5en.48xlarge** (8× H200 141 GB) — Capacity-Blocks-only; reservation
  fees ~$39.8 / $45.8 per hour.
- **p6-b200.48xlarge** (8× B200 180 GB) = **$113.9328/hr** list, GA since 2025-05-15
  in US West (Oregon); supports FP4 (NVFP4 / MXFP4) tensor cores natively.

## Single-GPU-by-VRAM map

| VRAM | GPU | AWS instance | $/hr |
|---|---|---|---:|
| 16 GB | T4 | g4dn.xlarge | $0.526 |
| 24 GB | A10G | g5.xlarge | $1.006 |
| 24 GB | L4 | g6.xlarge | $0.8048 |
| **48 GB** | L40S | **g6e.xlarge** | $1.861 |
| 80 GB | A100 | (no single-GPU AWS SKU; smallest = p4d.24xlarge 8× A100) | $32.77 |
| 80 GB | H100 | (no single-GPU AWS SKU; smallest = p5.48xlarge 8× H100) | $98.32 |
| 141 GB | H200 | (no single-GPU AWS SKU; smallest = p5e.48xlarge 8× H200) | ~$45.77 reserve |
| 180 GB | B200 | (no single-GPU AWS SKU; smallest = p6-b200.48xlarge 8× B200) | $113.93 |

Note: AWS does not sell **any** single-GPU SKU for A100/H100/H200/B200 — these are
multi-GPU-only chassis. The largest single-GPU AWS card is the **L40S 48 GB on g6e.xlarge**.

## Architectural compatibility for vLLM kernels

| GPU | sm | AWQ Marlin | FP8 native | MXFP4 native | NVLink |
|---|---|---|---|---|---|
| T4 (g4dn) | 75 | ✅ | ❌ | ❌ | ❌ |
| A10G (g5) | 86 | ✅ | ❌ | ❌ | ❌ |
| L4 (g6) | 89 | ✅ | ✅ | ❌ | ❌ |
| L40S (g6e) | 89 | ✅ | ✅ | ❌ | ❌ |
| A100 (p4d/p4de) | 80 | ✅ | ❌ | ❌ | ✅ NVSwitch |
| H100 (p5) | 90 | ✅ | ✅ | ✅ | ✅ NVSwitch |
| H200 (p5e/p5en) | 90 | ✅ | ✅ | ✅ | ✅ NVSwitch |
| B200 (p6-b200) | 100 | ✅ | ✅ | ✅ + FP4 | ✅ NVLink 5 |

**Practical implication**: gpt-oss-* MXFP4 weights run fastest on H100+; on A10G/L4/L40S
they dequantize to BF16 and double in size — often defeating the quantization gain.

## Pages updated on ingest

- [[hardware/aws-gpu-landscape]] — new master GPU table (replaces narrow multi-gpu page)
- [[hardware/multi-gpu-options]] — re-scoped to multi-GPU decision logic; defers
  pricing tables to the new master page
- [[comparisons/models-by-budget]] — new tier-by-tier $/hr → model-fit guide
- [[wiki/overview]] — broadened constraint section beyond g5.xlarge
