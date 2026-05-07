---
tags: [source, models, mistral, code, agent]
source_path: raw/devstral-small-cards.md
source_url: https://mistral.ai/news/devstral
ingested: 2026-05-07
last_updated: 2026-05-07
---

# Devstral — Mistral AI × All Hands AI agentic-coding line

Apache-2.0 dense models specifically post-trained for agentic coding scaffolds
(OpenHands, SWE-Agent). Devstral-Small-2507 is the cleanest "fits A10G + best
SWE-bench score" combination in the wiki.

## Key claims

- **Devstral-Small-2507** (May 2025): 24B dense, base = Mistral-Small-3.1-24B-Base.
  Apache-2.0. 128K ctx. SWE-bench Verified **53.6%** — #1 open-source at release.
- **Devstral-Small-2-24B-Instruct-2512** (Dec 2025): 24B dense. SWE-bench Verified **68.0%**.
- **Devstral-2-123B-Instruct-2512**: 123B dense, Apache-2.0, SWE-bench Verified **72.2%**.
- vLLM serving: `--tool-call-parser mistral --tokenizer_mode mistral
  --config_format mistral --load_format mistral`.
- Fit: 24B AWQ INT4 ~13 GB → comfortably single A10G; 123B AWQ INT4 ~65 GB →
  single H100 80 GB or 2× L40S TP=2 (g6e.12xlarge).

## Why Devstral matters for this wiki

- Apache-2.0 (no MNPL block like Codestral 22B) — **commercially deployable**.
- Dense (no MoE complexity) — predictable serving.
- Single A10G fit at INT4 — fits the original g5.xlarge constraint.
- Best open-source SWE-bench score for any model that fits g5.xlarge.

## Pages updated on ingest

- [[models/devstral-small]] — new (replaces Codestral-22B as the "Mistral coder"
  recommendation; Devstral is Apache-2.0 not MNPL)
- [[models/codestral-22b]] — superseded note added
- [[concepts/code-generation]] — refresh open-source SWE-bench leaderboard
- [[comparisons/tool-calling-models-on-a10g]] — added Devstral-Small-2507 row
- [[comparisons/models-by-budget]] — top recommendation in g5.xlarge ($1/hr) tier
