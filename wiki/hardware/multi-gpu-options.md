---
tags: [hardware, gpu, aws, multi-gpu]
last_updated: 2026-05-07
source_count: 0
---

# Multi-GPU options on AWS

When a model doesn't fit on a single A10G (24 GB), step up. See [[hardware/a10g-g5xlarge]] for the baseline.

## Decision tree

1. **Does the model fit on 1× A10G at AWQ INT4 with usable context?** → stay on g5.xlarge.
2. **Does it fit at INT8 or FP16 on 24 GB?** → stay on g5.xlarge (better quality than INT4).
3. **Need 2× the VRAM (~48 GB)?** → there's no 2× A10G g5 SKU; jump to **g5.12xlarge** (4× A10G = 96 GB) or consider a different family.
4. **Need 80–160 GB?** → **g5.48xlarge** (8× A10G = 192 GB) or step to A100/H100.
5. **Need fastest inference + NVLink + FP8?** → **p4d.24xlarge** (8× A100 40 GB) or **p5.48xlarge** (8× H100 80 GB).

## AWS GPU instance menu *(unverified — needs source)*

| Instance | GPUs | Total VRAM | Interconnect | On-demand $/hr | Notes |
|---|---|---|---|---|---|
| g5.xlarge | 1× A10G | 24 GB | — | ~$1.006 | baseline |
| g5.2xlarge | 1× A10G | 24 GB | — | ~$1.21 | more vCPU/RAM, **same GPU** |
| g5.12xlarge | 4× A10G | 96 GB | PCIe | ~$5.67 | TP=4 across PCIe |
| g5.24xlarge | 4× A10G | 96 GB | PCIe | ~$8.14 | more CPU/RAM than 12xl |
| g5.48xlarge | 8× A10G | 192 GB | PCIe | ~$16.29 | 70B models at FP16 fit here |
| p4d.24xlarge | 8× A100 (40 GB) | 320 GB | NVLink + EFA | ~$32.77 | NVLink, fast multi-GPU |
| p4de.24xlarge | 8× A100 (80 GB) | 640 GB | NVLink + EFA | ~$40.97 | 80 GB A100 |
| p5.48xlarge | 8× H100 (80 GB) | 640 GB | NVLink + EFA | ~$98+ | H100s, FP8 native |
| p5e.48xlarge | 8× H200 (141 GB) | ~1.1 TB | NVLink + EFA | very high | newest |

Numbers are us-east-1 on-demand and approximate; verify against the AWS pricing page.

## When does multi-GPU actually help?

- **Tensor parallelism (TP)** splits a model across N GPUs. Required when weights don't fit. Per-GPU compute drops linearly but communication overhead eats some of it (bad on PCIe-only g5; better on NVLink p4d/p5).
- **Pipeline parallelism (PP)** stages layers across GPUs. Better for very large models, worse for latency.
- **Disaggregated prefill/decode** (see [[infrastructure/nvidia-dynamo]]) splits the prefill workload from decode across separate GPU groups — only meaningful when you have ≥2 GPUs.

For models in this wiki:
- **Up to ~24B**: single A10G with AWQ INT4 is almost always cheapest. Multi-GPU is a quality (FP16) play, not a fit play.
- **32B**: AWQ INT4 fits 1× A10G but tight on context. 2× A10G via g5.12xlarge gives breathing room for FP16 or larger contexts.
- **70B**: needs g5.48xlarge (8× A10G) at INT4, p4d (8× A100 40 GB) at INT8/FP16, or p5 (8× H100) for best latency. See [[models/llama-3.3-70b-instruct]].

## Cost comparison: same model, different boxes

For a 32B AWQ-INT4 model (≈19 GB weights):

| Box | $/hr | $/month (24×7) | Notes |
|---|---|---|---|
| g5.xlarge | ~$1.01 | ~$734 | tight context (~7k) |
| g5.12xlarge (TP=2 unused) | ~$5.67 | ~$4,140 | wasteful — pays for 4 GPUs |
| g5.12xlarge (TP=4 FP16) | ~$5.67 | ~$4,140 | full FP16, much higher quality |

Spot pricing typically 30–70% off on-demand and is the realistic deployment for non-prod workloads.

## Related
- [[hardware/a10g-g5xlarge]]
- [[infrastructure/quantization]]
- [[infrastructure/nvidia-dynamo]]

## Sources
- (none yet)

## TODO / verify
- AWS EC2 G5/P4d/P5 current pricing
- Spot price history (AWS Spot Advisor)
- NVIDIA H100 vs A100 vs A10G inference benchmarks
