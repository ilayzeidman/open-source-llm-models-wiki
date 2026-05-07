---
tags: [hardware, gpu, aws, master-table]
last_updated: 2026-05-07
source_count: 3
---

# AWS GPU Landscape — full menu (May 2026)

The complete map of NVIDIA-GPU EC2 families that ship open-source LLMs at AWS,
with pricing, architecture, and per-tier model fit. Replaces the earlier narrow
[[hardware/multi-gpu-options]] page as the canonical price-and-spec reference.
The original [[hardware/a10g-g5xlarge]] page remains as the deep-dive on the
research's primary baseline.

## TL;DR for this research question

The "single GPU on AWS" universe has **5 SKUs**:

| SKU | GPU | VRAM | $/hr | Best for |
|---|---|---:|---:|---|
| g4dn.xlarge | T4 | 16 GB | $0.526 | 7B models, casual inference |
| **g5.xlarge** | A10G | 24 GB | $1.006 | **wiki baseline** — 14–24B AWQ INT4 |
| g6.xlarge | L4 | 24 GB | $0.8048 | cheap A10G alternative; FP8 native |
| **g6e.xlarge** | L40S | **48 GB** | $1.861 | **24–32B FP16 single-GPU; 70B INT4 via 8x** |
| (no AWS option) | A100/H100/H200/B200 | 40–180 GB | n/a as 1× | only sold in 8× P-class boxes |

That is, **AWS sells single-GPU SKUs only at 16 GB / 24 GB / 48 GB**. Above 48 GB
single-GPU you must rent an 8× P-class chassis.

## Full instance × price table (us-east-1 on-demand list)

[Source: [[sources/aws-extended-gpu-pricing-2026-05]]]

### G4dn — NVIDIA T4 (Turing sm_75, 16 GB, no FP8)

| Instance | GPUs | VRAM | $/hr | $/month |
|---|---:|---:|---:|---:|
| g4dn.xlarge | 1× T4 | 16 GB | $0.526 | ~$384 |
| g4dn.2xlarge | 1× T4 | 16 GB | $0.752 | ~$549 |
| g4dn.12xlarge | 4× T4 | 64 GB | $3.912 | ~$2,855 |
| g4dn.metal | 8× T4 | 128 GB | $7.824 | ~$5,711 |

### G5 — NVIDIA A10G (Ampere sm_86, 24 GB, no FP8)

| Instance | GPUs | VRAM | $/hr | $/month |
|---|---:|---:|---:|---:|
| g5.xlarge | 1× A10G | 24 GB | $1.006 | ~$734 |
| g5.2xlarge | 1× A10G | 24 GB | $1.212 | ~$885 |
| g5.4xlarge | 1× A10G | 24 GB | $1.624 | ~$1,185 |
| g5.8xlarge | 1× A10G | 24 GB | $2.448 | ~$1,787 |
| g5.16xlarge | 1× A10G | 24 GB | $4.096 | ~$2,990 |
| g5.12xlarge | 4× A10G | 96 GB | $5.672 | ~$4,140 |
| g5.24xlarge | 4× A10G | 96 GB | $8.144 | ~$5,945 |
| g5.48xlarge | 8× A10G | 192 GB | $16.288 | ~$11,890 |

### G6 — NVIDIA L4 (Ada sm_89, 24 GB, FP8 native)

| Instance | GPUs | VRAM | $/hr | $/month |
|---|---:|---:|---:|---:|
| g6.xlarge | 1× L4 | 24 GB | $0.8048 | ~$587 |
| g6.2xlarge | 1× L4 | 24 GB | $0.9776 | ~$714 |
| g6.4xlarge | 1× L4 | 24 GB | $1.3232 | ~$966 |
| g6.8xlarge | 1× L4 | 24 GB | $2.0144 | ~$1,471 |
| g6.16xlarge | 1× L4 | 24 GB | $3.3968 | ~$2,480 |
| g6.12xlarge | 4× L4 | 96 GB | $4.6024 | ~$3,360 |
| g6.24xlarge | 4× L4 | 96 GB | $6.6752 | ~$4,873 |
| g6.48xlarge | 8× L4 | 192 GB | $13.3504 | ~$9,746 |

### G6e — NVIDIA L40S (Ada sm_89, **48 GB**, FP8 native)

| Instance | GPUs | VRAM | $/hr | $/month |
|---|---:|---:|---:|---:|
| g6e.xlarge | 1× L40S | 48 GB | $1.861 | ~$1,358 |
| g6e.2xlarge | 1× L40S | 48 GB | $2.242 | ~$1,637 |
| g6e.4xlarge | 1× L40S | 48 GB | $3.004 | ~$2,193 |
| g6e.8xlarge | 1× L40S | 48 GB | $4.529 | ~$3,306 |
| g6e.16xlarge | 1× L40S | 48 GB | $7.578 | ~$5,532 |
| g6e.12xlarge | 4× L40S | 192 GB | $10.493 | ~$7,660 |
| g6e.24xlarge | 4× L40S | 192 GB | $15.066 | ~$11,000 |
| g6e.48xlarge | 8× L40S | 384 GB | $30.131 | ~$22,000 |

### P4 — NVIDIA A100 (Ampere sm_80, NVSwitch)

| Instance | GPUs | VRAM | $/hr | Notes |
|---|---:|---:|---:|---|
| p4d.24xlarge | 8× A100 40GB | 320 GB | $32.7726 | NVSwitch, EFA |
| p4de.24xlarge | 8× A100 80GB | 640 GB | $40.9657 | 2× memory of p4d |

### P5 — NVIDIA H100 / H200 (Hopper sm_90, NVSwitch, FP8 + MXFP4 native)

| Instance | GPUs | VRAM | $/hr | Notes |
|---|---:|---:|---:|---|
| p5.48xlarge | 8× H100 80GB | 640 GB | $98.32 | on-demand |
| p5e.48xlarge | 8× H200 141GB | ~1.13 TB | Capacity Blocks (~$39.8 reserve) | H200 |
| p5en.48xlarge | 8× H200 141GB | ~1.13 TB | Capacity Blocks (~$45.8 reserve) | H200 + better net |

### P6 — NVIDIA B200 (Blackwell sm_100, NVLink-5, FP4 native)

| Instance | GPUs | VRAM | $/hr | Notes |
|---|---:|---:|---:|---|
| p6-b200.48xlarge | 8× B200 180GB | 1.44 TB | **$113.9328** | GA 2025-05-15, US West (Oregon) primary |

## Architectural compatibility (critical for vLLM kernel choices)

| GPU (sm) | Marlin AWQ | FP8 native | MXFP4 native | NVLink |
|---|---|---|---|---|
| T4 (75) | ✅ | ❌ | ❌ | ❌ |
| A10G (86) | ✅ | ❌ | ❌ | ❌ |
| L4 (89) | ✅ | ✅ | ❌ | ❌ |
| L40S (89) | ✅ | ✅ | ❌ | ❌ |
| A100 (80) | ✅ | ❌ | ❌ | ✅ NVSwitch |
| H100 (90) | ✅ | ✅ | ✅ | ✅ NVSwitch |
| H200 (90) | ✅ | ✅ | ✅ | ✅ NVSwitch |
| B200 (100) | ✅ | ✅ | ✅ + FP4 | ✅ NVLink-5 |

[Source: [[sources/nvidia-l4-l40s-specs]], [[sources/aws-extended-gpu-pricing-2026-05]]]

**Practical implications**:

1. [[models/gpt-oss-20b]] / [[models/gpt-oss-120b]] use **MXFP4** — only run at native
   speed on H100/H200/B200. On A10G/L40S, vLLM dequantizes to BF16 (≈ 2× the VRAM).
2. **FP8 weights** (DeepSeek-V3.1, Kimi-K2, GLM-4.5 FP8 builds, gpt-oss native) require
   sm_89+ for hardware tensor cores. AWS's cheapest FP8 path is **g6.xlarge ($0.80) on L4**
   or **g6e.xlarge ($1.86) on L40S**.
3. **NVLink** matters for tensor-parallel-size ≥ 4 with frontier models — only on
   p4d/p4de/p5/p6. G-class multi-GPU is PCIe-Gen4 only and adds noticeable comms
   overhead at TP=8.

## Decision tree (single-node)

1. **Cost-minimal, model ≤ 8B**: g4dn.xlarge ($0.526) at AWQ INT4 → [[models/qwen3-8b]]-class, [[models/granite-3.1-8b]], [[models/hermes-3-llama-3.1-8b]]
2. **Wiki baseline, ≤ 24B**: g5.xlarge ($1.006) → [[models/qwen3-coder-30b-a3b]], [[models/devstral-small]], [[models/granite-3.1-8b]]
3. **Cheaper baseline, FP8 OK**: g6.xlarge ($0.80) — beware lower memory bandwidth
4. **One GPU at 48 GB**: g6e.xlarge ($1.861) → [[models/devstral-small]] FP16, [[models/qwen3-32b]] AWQ comfortable, [[models/qwen3-coder-30b-a3b]] FP8
5. **70B AWQ single-node**: g5.48xlarge ($16.29) or g6e.12xlarge ($10.49) TP=4
6. **GLM-4.5-Air-class (~106B MoE)**: g6e.12xlarge ($10.49) FP8 TP=2 / 4 → [[models/glm-4.5-air]]
7. **120B–123B dense**: g6e.12xlarge AWQ TP=4 → [[models/devstral-small]] (123B variant)
8. **Llama-4-Scout (109B/17B), Maverick**: g6e.48xlarge ($30.13) or p4d.24xlarge ($32.77)
9. **Frontier MoE (DeepSeek-V3.1, Qwen3-Coder-480B, Kimi-K2)**: p5e/p5en or p6-b200
10. **MXFP4 native (gpt-oss)**: p5.48xlarge slice or p6-b200.48xlarge

## Spot vs on-demand

Spot pricing is typically 30–70% off list. For non-prod, spot on g5/g6/g6e is the
realistic deployment cost. Spot on p5/p6 is rare and time-limited; for production
agentic workloads on those tiers, plan for Capacity Blocks or 1y Savings Plan
commits.

## Related

- [[hardware/a10g-g5xlarge]] — deep dive on the wiki baseline GPU
- [[hardware/multi-gpu-options]] — multi-GPU decision logic (TP/PP/disagg)
- [[infrastructure/quantization]] — which quants run where
- [[comparisons/models-by-budget]] — tier-by-tier model recommendations

## Sources

- [[sources/aws-extended-gpu-pricing-2026-05]]
- [[sources/nvidia-l4-l40s-specs]]
- [[sources/nvidia-a10g-specs]]
