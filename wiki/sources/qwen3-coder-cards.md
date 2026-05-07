---
tags: [source, models, code, tool-calling, qwen]
source_path: raw/qwen3-coder-cards.md
source_url: https://qwenlm.github.io/blog/qwen3-coder/
ingested: 2026-05-07
last_updated: 2026-05-07
---

# Qwen3-Coder — HF cards + Qwen blog + vLLM recipe

The successor to Qwen2.5-Coder, released as agentic-coding-tuned MoE in 2025-Q3.
This is the most important new code+tool model since the wiki's last major ingest
because it ships the missing `qwen3_coder` parser path that fixes the Qwen2.5-Coder
flaky-`hermes`-parser problem documented in [[wiki/overview]].

## Key claims

- **Qwen3-Coder-30B-A3B-Instruct**: 31B total / **3.3B active** MoE, Apache-2.0,
  256K native context (1M with YaRN). Footprint: BF16 ~62 GB; **AWQ INT4 ~17–18 GB
  fits A10G/L4/L40S single GPU**.
- **Qwen3-Coder-480B-A35B-Instruct**: 480B total / 35B active MoE, Apache-2.0.
  **66.5% Pass@1 on SWE-bench Verified** (Qwen blog). Needs 8× H100/H200/B200.
- **vLLM tool-call parser `qwen3_coder`** (mainline since vLLM 0.10.0) — the
  successor to `hermes` for the Qwen-Coder family. Resolves the JSON-fences-vs-
  `<tool_call>` parser bug documented for Qwen2.5-Coder.
- Non-thinking mode only — these are agentic models, not reasoning models.
- Qwen3-Coder-30B-A3B SWE-bench Verified ~50.3% — beats every dense ≤32B open model.

## Pages updated on ingest

- [[models/qwen3-coder-30b-a3b]] — new (A10G/L40S sweet spot for code+tools)
- [[models/qwen3-coder-480b]] — new (multi-node flagship)
- [[models/qwen2.5-coder-14b]], [[models/qwen2.5-coder-32b]] — superseded notes
- [[concepts/code-generation]] — refresh top-of-leaderboard
- [[infrastructure/vllm]] — `qwen3_coder` parser added
- [[comparisons/models-by-budget]] — top recommendation in $1.86–$30/hr tiers
- [[comparisons/tool-calling-models-on-a10g]] — added Qwen3-Coder-30B-A3B row
