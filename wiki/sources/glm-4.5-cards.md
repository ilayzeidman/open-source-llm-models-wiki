---
tags: [source, models, glm, agent, mix-of-experts]
source_path: raw/glm-4.5-cards.md
source_url: https://github.com/zai-org/GLM-4.5
ingested: 2026-05-07
last_updated: 2026-05-07
---

# GLM-4.5 / GLM-4.5-Air — Z.ai "ARC" agentic foundation models

The "Agentic, Reasoning, and Coding" line from Z.ai (formerly Zhipu). MIT license
for both sizes. GLM-4.5-Air is the most under-appreciated mid-tier open-source
agent model and a strong fit for AWS g6e multi-GPU.

## Key claims

- **GLM-4.5**: 355B total / 32B active MoE. MIT. SWE-bench Verified **64.2%**,
  TAU-Bench **70.1%**.
- **GLM-4.5-Air**: 106B total / 12B active MoE. MIT. Aggregate score 59.8 across
  12 benchmarks (vs 63.2 for full GLM-4.5).
- vLLM tool-call parser: **`glm45`** (mainline since 0.10.x).
- Fit: GLM-4.5-Air FP8 ≈ 106 GB → **does NOT fit 2× L40S (96 GB)**. Smallest
  single-node FP8 fit is **TP=4 on g6e.12xlarge** (4× L40S = 192 GB, ~27 GB/GPU).
  AWQ INT4 ≈ 53 GB exceeds a single L40S 48 GB by ~5 GB → also needs ≥2 GPUs;
  cleanest is INT4 TP=4 on g6e.12xlarge.

## Pages updated on ingest

- [[models/glm-4.5-air]] — new (g6e/p4d sweet spot)
- [[infrastructure/vllm]] — `glm45` parser added
- [[concepts/tool-selection]] — TAU-Bench 70.1 is among open-source leaders
- [[comparisons/models-by-budget]] — top recommendation in $10–$30/hr tier
