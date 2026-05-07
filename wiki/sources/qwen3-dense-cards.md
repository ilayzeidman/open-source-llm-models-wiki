---
tags: [source, models, generalist, qwen, reasoning]
source_path: raw/qwen3-dense-cards.md
source_url: https://qwenlm.github.io/blog/qwen3/
ingested: 2026-05-07
last_updated: 2026-05-07
---

# Qwen3 dense models — Qwen3-32B / 14B / 8B

Hybrid reasoning generalists from the Qwen3 release (Apache-2.0). The dense
counterpart to the Qwen3-Coder MoE line. Hybrid reasoning means a single model
toggles between explicit `<think>` chain-of-thought and direct response via
`enable_thinking` or `/think` `/no_think` soft tags.

## Key claims

- **Qwen3-32B**: 32.8B params, Apache-2.0, 32K native ctx (131K with YaRN).
  AWQ INT4 ~18 GB → A10G fit (tight ctx) or single L40S comfortable.
- **Qwen3-14B**: 14B dense, Apache-2.0, AWQ INT4 ~9 GB → A10G generous fit.
- **Qwen3-8B**: 8B dense, Apache-2.0, AWQ INT4 ~5 GB → fits g4dn.xlarge / g6.xlarge.
- BFCL v3 (235B-A22B flagship): 70.8 — Qwen3 is on the BFCL leaderboard for the
  first time, unlike Qwen2.5-Coder.
- vLLM: `--enable-reasoning --reasoning-parser deepseek_r1` for the `<think>` block,
  plus `--tool-call-parser hermes` for tool calls (Qwen3 dense uses the standard
  Hermes-shaped output).

## Pages updated on ingest

- [[models/qwen3-32b]] — new
- [[concepts/tool-selection]] — Qwen3 BFCL v3 = 70.8 (flagship)
- [[concepts/benchmarks]] — note BFCL v3 + v4 are now leading metrics for tool calling
- [[comparisons/tool-calling-models-on-a10g]] — added Qwen3-32B row
