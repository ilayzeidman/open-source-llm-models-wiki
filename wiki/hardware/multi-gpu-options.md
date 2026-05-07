---
tags: [hardware, gpu, aws, multi-gpu]
last_updated: 2026-05-07
source_count: 1
---

# Multi-GPU options on AWS

When a model doesn't fit on a single A10G (24 GB), step up. See [[hardware/a10g-g5xlarge]] for the baseline.

## Decision tree

1. **Does the model fit on 1× A10G at AWQ INT4 with usable context?** → stay on g5.xlarge.
2. **Does it fit at INT8 or FP16 on 24 GB?** → stay on g5.xlarge (better quality than INT4).
3. **Need 2× the VRAM (~48 GB)?** → there's no 2× A10G g5 SKU; jump to **g5.12xlarge** (4× A10G = 96 GB) or consider a different family.
4. **Need 80–160 GB?** → **g5.48xlarge** (8× A10G = 192 GB), or step to A100/H100 for NVLink and FP8.
5. **Need fastest inference + NVLink + FP8?** → **p4d.24xlarge** (8× A100 40 GB) or **p5.48xlarge** (8× H100 80 GB).

## AWS GPU instance menu (us-east-1, on-demand list pricing)

[Source: [[sources/aws-ec2-pricing-2026-05]]]

| Instance | GPUs | Total VRAM | Interconnect | $/hr | Notes |
|---|---|---|---|---:|---|
| g5.xlarge | 1× A10G | 24 GB | — | $1.006 | baseline |
| g5.2xlarge | 1× A10G | 24 GB | — | $1.212 | more vCPU/RAM, **same GPU** |
| g5.4xlarge | 1× A10G | 24 GB | — | $1.624 | |
| g5.8xlarge | 1× A10G | 24 GB | — | $2.448 | |
| g5.16xlarge | 1× A10G | 24 GB | — | $4.096 | |
| g5.12xlarge | 4× A10G | 96 GB | PCIe Gen4 | $5.672 | TP=4 across PCIe |
| g5.24xlarge | 4× A10G | 96 GB | PCIe Gen4 | $8.144 | more CPU/RAM than 12xl |
| g5.48xlarge | 8× A10G | 192 GB | PCIe Gen4 | $16.288 | 70B at AWQ INT4 fits |
| p4d.24xlarge | 8× A100 (40 GB) | 320 GB | NVLink + EFA | $32.7726 | NVLink, fast multi-GPU |
| p4de.24xlarge | 8× A100 (80 GB) | 640 GB | NVLink + EFA | $40.9657 | 80 GB A100 |
| p5.48xlarge | 8× H100 (80 GB) | 640 GB | NVLink + EFA | $98.32 | H100s, FP8 native |
| p5e.48xlarge | 8× H200 (141 GB) | ~1.1 TB | NVLink + EFA | Capacity-Blocks-only | $39.799/hr reservation fee (US) |
| p5en.48xlarge | 8× H200 (141 GB) | ~1.1 TB | NVLink + EFA | Capacity-Blocks-only | $45.768/hr reservation fee |

> ⚠ Source caveat: some pricing aggregators (e.g. vantage.sh) show lower P4/P5 numbers (~$22 p4d, ~$55 p5.48xl) which appear to be Savings-Plan / Reserved rates, not list on-demand. Confirm against the AWS pricing API for canonical figures. [Source: [[sources/aws-ec2-pricing-2026-05]]]

## When does multi-GPU actually help?

- **Tensor parallelism (TP)** splits a model across N GPUs. Required when weights don't fit. Per-GPU compute drops linearly but communication overhead eats some of it (notable on PCIe-only g5; minor on NVLink p4d/p5).
- **Pipeline parallelism (PP)** stages layers across GPUs. Better for very large models, worse for latency.
- **Disaggregated prefill/decode** (see [[infrastructure/nvidia-dynamo]]) splits the prefill workload from decode across separate GPU groups — only meaningful when you have ≥2 GPUs.

For models in this wiki:
- **Up to ~24B**: single A10G with AWQ INT4 is almost always cheapest. Multi-GPU is a quality (FP16) play, not a fit play.
- **32B**: AWQ INT4 fits 1× A10G but tight on context. 2× A10G via g5.12xlarge gives breathing room for FP16 or larger contexts.
- **70B**: needs g5.48xlarge (8× A10G) at INT4, p4d (8× A100 40 GB) at INT8/FP16, or p5 (8× H100) for best latency. See [[models/llama-3.3-70b-instruct]].

## Cost comparison: same model, different boxes

For a 32B AWQ-INT4 model (≈19 GB weights):

| Box | $/hr | $/month (24×7) | Notes |
|---|---:|---:|---|
| g5.xlarge | $1.006 | ~$734 | tight context (~7k) |
| g5.12xlarge (TP=2 unused) | $5.672 | ~$4,140 | wasteful — pays for 4 GPUs |
| g5.12xlarge (TP=4 FP16) | $5.672 | ~$4,140 | full FP16, much higher quality |

Spot pricing typically 30–70% off on-demand and is the realistic deployment for non-prod workloads.

## Related
- [[hardware/a10g-g5xlarge]]
- [[infrastructure/quantization]]
- [[infrastructure/nvidia-dynamo]]

## Sources
- [[sources/aws-ec2-pricing-2026-05]]
