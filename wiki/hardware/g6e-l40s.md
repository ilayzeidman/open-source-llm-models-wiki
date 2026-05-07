---
tags: [hardware, gpu, aws, l40s]
last_updated: 2026-05-07
source_count: 2
---

# AWS g6e — single L40S 48 GB

The most under-used AWS GPU SKU for open-source LLM serving. **g6e.xlarge at
$1.861/hr is the cheapest 48 GB single-GPU box on AWS** — and unlocks model fits
that are impossible on A10G or L4 (24 GB).

## Specs (per L40S)

- 48 GB GDDR6
- 864 GB/s memory bandwidth (vs A10G 600 GB/s, L4 300 GB/s)
- ~362 TF FP16 dense, ~733 TF FP8 dense
- Ada Lovelace, sm_89 — **FP8 tensor cores native**, no MXFP4
- 350 W TDP

## What "48 GB single-GPU" unlocks

[Source: [[sources/aws-extended-gpu-pricing-2026-05]], [[sources/nvidia-l4-l40s-specs]]]

| Model | A10G 24 GB fit | L40S 48 GB fit | Why it matters |
|---|---|---|---|
| [[models/qwen3-coder-30b-a3b]] | AWQ INT4, ~30K ctx | **FP8, full 256K ctx** | quality + ctx headroom |
| [[models/qwen3-32b]] | AWQ INT4, ~7K ctx | **AWQ INT4, ~80K ctx** | usable agent loops |
| [[models/qwen2.5-coder-32b]] | AWQ INT4, ~7K ctx | **FP8 single-GPU** | full quality |
| [[models/devstral-small]] (24B) | AWQ INT4 fits | **FP8 fits** | full quality |
| [[models/mistral-small-24b]] | AWQ INT4 fits | **FP16 fits with KV** | best quality single-GPU |

## Single-GPU lineup

[Source: [[sources/aws-extended-gpu-pricing-2026-05]]]

| Instance | vCPUs | Mem (GiB) | $/hr |
|---|---:|---:|---:|
| g6e.xlarge | 4 | 32 | $1.861 |
| g6e.2xlarge | 8 | 64 | $2.242 |
| g6e.4xlarge | 16 | 128 | $3.004 |
| g6e.8xlarge | 32 | 256 | $4.529 |
| g6e.16xlarge | 64 | 512 | $7.578 |

For pure single-GPU LLM inference, g6e.xlarge is enough — bigger sizes only help
if you also need lots of CPU RAM for tokenizer/preprocessing or local NVMe for
model warmup.

## Multi-GPU g6e variants (PCIe Gen4, no NVLink)

| Instance | GPUs | Total VRAM | $/hr |
|---|---|---:|---:|
| g6e.12xlarge | 4× L40S | 192 GB | $10.493 |
| g6e.24xlarge | 4× L40S | 192 GB | $15.066 |
| g6e.48xlarge | 8× L40S | 384 GB | $30.131 |

g6e.48xlarge is the cheapest single-node fit for 70B AWQ INT4 with comfortable ctx,
and the smallest box that can serve [[models/llama-4-scout]] at INT4 TP=8.

## When to pick g6e.xlarge over g5.xlarge

- You need **48 GB of VRAM** on a single GPU (32B FP8, 24B FP16, or 30B-A3B FP8 with full ctx).
- You want **FP8 native** (DeepSeek-V3-quant style, gpt-oss BF16 dequant fit).
- You need **higher memory bandwidth** for long-context decode (864 vs 600 GB/s).
- Your workload is multi-tenant / batched — L40S has more compute headroom.

## When to stick with g5.xlarge

- You only run ≤ 24B AWQ INT4 with short ctx — the extra $0.85/hr buys nothing.
- You want the cheapest commercial-friendly LLM box and the model already fits.

## Related

- [[hardware/aws-gpu-landscape]] — full AWS GPU menu
- [[hardware/a10g-g5xlarge]] — A10G deep dive
- [[infrastructure/quantization]] — FP8 unlocked on Ada

## Sources

- [[sources/aws-extended-gpu-pricing-2026-05]]
- [[sources/nvidia-l4-l40s-specs]]
