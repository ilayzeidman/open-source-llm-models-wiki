# BFCL Leaderboard Snapshot (May 2026)

source_url: https://gorilla.cs.berkeley.edu/leaderboard.html
fetched: 2026-05-07

## Status of the live page

The official BFCL leaderboard at https://gorilla.cs.berkeley.edu/leaderboard.html is now **BFCL V4** (last commit `f7cf735`, last updated 2026-04-12). The live page is JavaScript-rendered and does not return scores via raw HTML fetch. Underlying data lives in the `gorilla` repo in CSVs:

- `score/data_overall.csv`
- `score/data_live.csv`
- `score/data_non_live.csv`
- `score/data_multi_turn.csv`

(See `https://github.com/ShishirPatil/gorilla/tree/main/berkeley-function-call-leaderboard`.)

## BFCL V3 mirror (via llm-stats.com)

Sourced from https://llm-stats.com/benchmarks/bfcl-v3 (snapshot 2026-05-07). This mirror only has 18 models from a more recent submission window — most "old" 2024 candidates (Qwen2.5-Coder, Llama-3.1, xLAM, Hermes, etc.) are no longer on this mirror's top 18.

| Rank | Model | Org | BFCL v3 Score |
|------|-------|-----|---------------|
| 1 | GLM-4.5 | Zhipu AI | 0.778 |
| 2 | GLM-4.5-Air | Zhipu AI | 0.764 |
| 3 | LongCat-Flash-Thinking | Meituan | 0.744 |
| 4 | Qwen3-Next-80B-A3B-Thinking | Alibaba/Qwen | 0.720 |
| 5 | Qwen3-VL-235B-A22B-Thinking | Alibaba/Qwen | 0.719 |
| 5 | Qwen3-235B-A22B-Thinking-2507 | Alibaba/Qwen | 0.719 |
| 7 | Qwen3-VL-32B-Thinking | Alibaba/Qwen | 0.717 |
| 8 | Qwen3-235B-A22B-Instruct-2507 | Alibaba/Qwen | 0.709 |
| 9 | Qwen3-Next-80B-A3B-Instruct | Alibaba/Qwen | 0.703 |
| 10 | Qwen3-VL-32B-Instruct | Alibaba/Qwen | 0.702 |
| 11 | Qwen3-Coder-480B-A35B-Instruct | Alibaba/Qwen | 0.687 |
| 12 | Qwen3-VL-30B-A3B-Thinking | Alibaba/Qwen | 0.686 |
| 13 | Qwen3-VL-235B-A22B-Instruct | Alibaba/Qwen | 0.677 |
| 14 | Qwen3-VL-4B-Thinking | Alibaba/Qwen | 0.673 |
| 15 | Qwen3-VL-8B-Instruct | Alibaba/Qwen | 0.663 |
| 15 | Qwen3-VL-30B-A3B-Instruct | Alibaba/Qwen | 0.663 |
| 17 | Qwen3-VL-4B-Instruct | Alibaba/Qwen | 0.633 |
| 18 | Qwen3-VL-8B-Thinking | Alibaba/Qwen | 0.630 |

## Alternate BFCL v3 mirror (pricepertoken.com)

Sourced from https://pricepertoken.com/leaderboards/benchmark/bfcl-v3 (snapshot 2026-05-07). Different set of evaluated models; includes some older.

| Rank | Model | Provider | BFCL v3 Score |
|------|-------|----------|---------------|
| 1 | GLM-4.5 Thinking | Z AI | 76.7 |
| 2 | Qwen3-32B Thinking | Alibaba | 75.7 |
| 3 | Qwen3-32B | Alibaba | 75.7 |
| 4 | Qwen3 Max | Alibaba | 74.9 |
| 6 | GLM-4.7-Flash Thinking | Z AI | 74.6 |
| 7 | GLM-4.7-Flash | Z AI | 74.6 |
| 8 | GLM-4.5 Air | Z AI | 69.1 |
| 9 | Nova Pro 1.0 | Amazon | 67.9 |
| 10 | Kimi K2.5 Thinking | Moonshot | 64.5 |
| 11 | Kimi K2.5 | Moonshot | 64.5 |
| 12 | INTELLECT-3 | Prime Intellect | 63.5 |
| 13 | Llama 4 Scout | Meta | 55.7 |
| 14 | Gemini 3 Flash Preview Thinking | Google | 53.5 |
| 15 | MiniMax M1 | MiniMax | 47.8 |
| 17 | Nemotron 3 Nano 30B A3B Thinking | NVIDIA | 41.6 |
| 18 | Nemotron 3 Nano 30B A3B | NVIDIA | 41.6 |
| 19 | Phi-4 | Microsoft | 40.8 |
| 20 | Claude Opus 4 Thinking | Anthropic | 25.3 |
| 21 | Claude Opus 4 | Anthropic | 25.3 |
| 22 | Kimi K2 0711 | Moonshot | 25.3 |

## Historical / per-model BFCL scores from primary sources

Collected from model cards, papers, and announcements for the 13 wiki models:

| Model | BFCL Score | Version | Source |
|-------|-----------|---------|--------|
| Llama-3.1-8B-Instruct | 76.1 | v2 (Meta self-report) | meta-llama eval_details |
| Llama-3.3-70B-Instruct | 77.3 | v2 | Galileo / community |
| Hermes-3-Llama-3.1-8B | ~91 (well-formed JSON rate, NOT overall acc) | v3 | Nous community blogs |
| xLAM-7b-fc-r | 88.24 (overall acc) | v1 (07/2024) | Salesforce model card |
| xLAM-1b-fc-r | 78.94 | v1 | Salesforce model card |
| xLAM-2-70b-fc-r | SOTA on BFCL & τ-bench (56.2% τ-bench) | v3 | Salesforce model card |
| Functionary-medium-v3.1 | 88.88 | not specified | MeetKai GitHub |
| Functionary-small-v3.2 | 82.82 | not specified | MeetKai GitHub |
| Functionary-small-v3.1 | 82.53 | not specified | MeetKai GitHub |
| Granite-3.1-8B-Instruct | (not on model card) | — | — |
| Granite-4.1-8B-Instruct | 68.27 | v3 | IBM blog |
| Phi-4 | 40.8 | v3 | pricepertoken mirror |
| Qwen2.5-Coder-32B-Instruct | not directly published; was on early BFCL | — | — |
| Qwen2.5-Coder-7B/14B-Instruct | not directly published | — | — |
| DeepSeek-Coder-V2-Lite-Instruct | not directly published | — | — |
| Mistral-Small-3 / 3.1 | not directly published; "best-in-class agentic" claim | — | — |
| Codestral-22B v0.1 | not directly published | — | — |

## Key flags

- BFCL V1 → V2 → V3 → V4 are NOT directly comparable. V1 was function-call AST + executable. V2 added live data. V3 added multi-turn (1000+ entries). V4 added agentic / web-search / memory / format-sensitivity.
- Most reported scores are at FP16 / BF16. The leaderboard page does not always state quantization; self-hosted entries via vLLM/sglang are typically BF16.
- Scores from model self-reports (Meta, Salesforce, MeetKai, IBM) may be on different BFCL commits than the official leaderboard.
