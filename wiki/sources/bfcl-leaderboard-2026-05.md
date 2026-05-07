---
tags: [source, benchmarks, tool-calling]
source_path: raw/bfcl-leaderboard-2026-05.md, raw/bfcl-methodology.md, raw/bfcl-overview-2026.md
source_url: https://gorilla.cs.berkeley.edu/leaderboard.html
ingested: 2026-05-07
last_updated: 2026-05-07
---

# BFCL — Berkeley Function-Calling Leaderboard (2026-05 snapshot)

The standard leaderboard for LLM tool-calling capability. Hosted by the Gorilla team at UC Berkeley.

## Key claims

### Methodology versions

| Version | Released | Adds |
|---|---|---|
| V1 | Feb 2024 | Single-turn AST + executable tests |
| V2 | Sep 2024 | Live data, parallel calls |
| V3 | 2024-Q4 | Multi-turn (Multi-Turn Base 200 + Augmented 800), hallucination/irrelevance categories |
| V4 | 2025/2026 | Agentic web-search, memory, format sensitivity |

V1 is saturated; **V3 multi-turn is the current discriminator**. Scoring uses AST-based parameter validation plus state-based / response-based checks for multi-turn.

### Per-model scores for the 13 wiki candidates

| Model | Score | BFCL Version | Caveat |
|---|---|---|---|
| Qwen2.5-Coder (7B/14B/32B) | **not on leaderboard** | — | Absent from BFCL SUPPORTED_MODELS as of 2026-04-12 |
| Llama-3.1-8B-Instruct | 76.1 | v2 | Meta self-report (eval_details) |
| Llama-3.3-70B-Instruct | 77.3 | v2 | Meta self-report |
| DeepSeek-Coder-V2-Lite-Instruct | not on leaderboard | — | Only DeepSeek-V3.2-Exp listed |
| Mistral-Small (3 / 3.1 / 3.2) | not on leaderboard | — | Mistral-Small-2506 (=3.2) on supported list, no published score |
| Codestral-22B v0.1 | not on leaderboard | — | — |
| Phi-3-Medium | not on leaderboard | — | — |
| Phi-4 | 40.8 | v3 | pricepertoken mirror, 2026-05-07 |
| Granite-3.1-8B-Instruct | not in IBM model card | — | — |
| Granite-4.1-8B-Instruct (newer) | 68.27 | v3 | IBM blog ("Granite family") |
| Hermes-3-Llama-3.1-8B | ~91% well-formed JSON rate | v3 | **NOT BFCL overall accuracy** — different metric. Don't quote as comparable. |
| xLAM-7b-fc-r | 88.24 (overall acc) | **v1 (07/2024)** | Salesforce model card. Pre-V3, not directly comparable. |
| Llama-xLAM-2-8b-fc-r | "SOTA on BFCL" qualitative | v3 | Salesforce model card; specific number not extractable |
| xLAM-2-70b-fc-r | "SOTA"; τ-bench 56.2% pass@1 | v3 | Salesforce — beats GPT-4o 52.9%, approaches Claude 3.5 Sonnet 60.1% |
| Functionary-medium-v3.1 | 88.88 | v2-era | MeetKai GitHub README |
| Functionary-small-v3.2 | 82.82 | v2-era | MeetKai GitHub README |
| Functionary-small-v3.1 | 82.53 | v2-era | MeetKai GitHub README |

### Top of the V3 leaderboard for context (May 2026)
- GLM-4.5: 0.778
- Qwen3-Next-80B-A3B-Thinking: 0.720
- Qwen3-235B-A22B-Instruct-2507: 0.709

### Important caveats

1. **All public BFCL scores are FP16/BF16.** No public AWQ/INT4 evaluations. Quantization may shift these scores; flag as a known unknown.
2. **Version drift** — Llama 3.x scores are V2; xLAM-7b-fc-r is V1; current public leaderboard is V3/V4. **Not directly comparable** across versions.
3. **Hermes-3 ambiguity** — community blogs report "~91% on BFCL" but this is the well-formed-JSON rate, not BFCL overall accuracy.
4. **Qwen2.5-Coder absent** despite strong claimed tool calling — either deprecated or never submitted. Major data gap for the wiki.
5. **JS-rendered leaderboard** — gorilla.cs.berkeley.edu's tables are client-rendered. Mirrors used: llm-stats.com/benchmarks/bfcl-v3, pricepertoken.com.

## Pages updated on ingest
- [[concepts/benchmarks]]
- [[concepts/tool-selection]]
- All [[wiki/comparisons/tool-calling-models-on-a10g|model pages]]
