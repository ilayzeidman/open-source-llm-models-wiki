---
tags: [models, glm, agent, mix-of-experts, reasoning]
params: 106B
active_params: 12B
license: MIT
context: 128K
release_date: 2025-08-01
last_updated: 2026-05-07
source_count: 1
---

# GLM-4.5-Air

MIT-licensed MoE (106B / 12B active) from Z.ai's "ARC" line (Agentic, Reasoning,
Coding). The most credible mid-tier open-source agent model — hits a sweet spot
between Qwen3-Coder-30B-A3B (smaller, A10G fit) and Qwen3-Coder-480B / DeepSeek-V3.1
(multi-node).

## Fit on AWS GPUs

| Box | GPUs | Quant | Weights | Verdict |
|---|---|---|---:|---|
| g6e.xlarge | 1× L40S 48GB | AWQ INT4 | ~53 GB | ❌ exceeds 48 GB by ~5 GB |
| g6e.12xlarge | 4× L40S | FP8 (TP=2 unused) | ~106 GB | ✅ fits TP=2, 96 GB headroom |
| g6e.12xlarge | 4× L40S | AWQ INT4 (TP=4) | ~13 GB/GPU | ✅ generous ctx |
| g6e.48xlarge | 8× L40S | FP8 (TP=8) | ~13 GB/GPU | ✅ best L40S option |
| p4d.24xlarge | 8× A100 40GB | AWQ INT4 (TP=8) | ~7 GB/GPU | ✅ NVLink fast |
| p5.48xlarge slice | 2× H100 | FP8 (TP=2) | ~53 GB/GPU | ✅ but pays for 8 |

## Strengths

- **64.2% SWE-bench Verified** (parent GLM-4.5; Air slightly lower but on the
  same trajectory).
- **TAU-Bench 70.1%** — among the strongest open-source tool-calling benchmarks.
- MIT license — fully commercial.
- vLLM mainline `glm45` parser since 0.10.x.
- 12B active → respectable decode throughput despite the 106B total.

## Weaknesses

- Doesn't fit single-GPU 48 GB at FP8 — needs ≥2 GPUs. Smallest single-node fit
  is g6e.12xlarge ($10.49/hr) or 2× L40S manually (no AWS SKU smaller than 4×).
- TAU-Bench / SWE-bench Verified for GLM-4.5-**Air** specifically (not the 355B
  parent) less widely benchmarked.

## vLLM serving

```
vllm serve zai-org/GLM-4.5-Air \
  --tensor-parallel-size 2 \
  --enable-auto-tool-choice --tool-call-parser glm45 \
  --max-model-len 32768
```

## Cost

| Box | $/hr | Quant | Note |
|---|---:|---|---|
| g6e.12xlarge | $10.49 | FP8 TP=2 | recommended sweet spot |
| g6e.48xlarge | $30.13 | FP8 TP=8 | full quality, scaled up |
| p4d.24xlarge | $32.77 | AWQ INT4 TP=8 | NVLink + Marlin |

## Related

- [[models/qwen3-coder-30b-a3b]] (smaller cousin, A10G fit)
- [[models/deepseek-v3.1]] (next tier up)
- [[hardware/aws-gpu-landscape]]

## Sources

- [[sources/glm-4.5-cards]]

## TODO / verify

- Independent SWE-bench / TAU-Bench scores for **Air specifically** (most numbers
  in the wild are for the 355B parent).
