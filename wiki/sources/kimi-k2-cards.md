---
tags: [source, models, moonshot, agent, mix-of-experts]
source_path: raw/kimi-k2-cards.md
source_url: https://moonshotai.github.io/Kimi-K2/
ingested: 2026-05-07
last_updated: 2026-05-07
---

# Kimi K2 / K2-Instruct / K2-Thinking — Moonshot AI

The first 1T-parameter open-weight model with credible SWE-bench performance.
Modified MIT license (open-weight; training data/code not released).

## Key claims

- **Kimi-K2-Instruct**: 1T total / 32B active (384 experts, 8 selected).
  Modified MIT. 128K ctx. **Block-FP8 native**.
- SWE-bench Verified: **65.8%** single-attempt, 71.6% with parallel TTC scoring.
- K2.5: 76.8% SWE-bench Verified. K2.6: **80.2%** SWE-bench Verified.
- Fit: FP8 ≈ 1 TB → fits **8× H200 single-node (p5e/p5en, 1.128 TB)** with headroom,
  or **8× B200 on p6-b200.48xlarge** comfortably. Will NOT fit p5.48xlarge (8× H100
  = 640 GB) at FP8.
- vLLM: `vllm serve moonshotai/Kimi-K2-Instruct`; tool calling supported with
  autonomous invocation. Recommended T=0.6.

## Pages updated on ingest

- [[models/kimi-k2]] — new (frontier open-source SWE-bench)
- [[concepts/code-generation]] — note K2.6 80.2% SWE-bench Verified
- [[comparisons/models-by-budget]] — top of P5e/P6 tier
