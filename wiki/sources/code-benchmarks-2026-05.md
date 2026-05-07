---
tags: [source, benchmarks, code]
source_path: raw/swebench-verified-2026-05.md, raw/swebench-methodology.md, raw/livecodebench-2026-05.md, raw/livecodebench-methodology.md, raw/aider-leaderboard-2026-05.md, raw/evalplus-leaderboard-2026-05.md
source_url: https://www.swebench.com/, https://livecodebench.github.io/, https://aider.chat/docs/leaderboards/, https://evalplus.github.io/leaderboard.html
ingested: 2026-05-07
last_updated: 2026-05-07
---

# Code-generation leaderboards (2026-05 snapshot)

SWE-bench Verified, LiveCodeBench, Aider, and EvalPlus (HumanEval+ / MBPP+) — the canonical code-evaluation leaderboards as of May 2026.

## Key claims

### Methodology summary

| Benchmark | Measures | Saturation |
|---|---|---|
| **HumanEval / HumanEval+** (EvalPlus) | 164 Python problems; EvalPlus adds aggressive tests | Saturated >92% for top models |
| **MBPP / MBPP+** (EvalPlus) | ~378 sanitized Python problems | Same |
| **LiveCodeBench** | Algorithmic problems; date-stamped, contamination-resistant | Active, primary differentiator |
| **SWE-bench Verified** | 500 human-validated GitHub issues; agent-scored on FAIL_TO_PASS + PASS_TO_PASS | Active; ⚠ contamination flag |
| **Aider polyglot** | 225 Exercism, 6 languages — current standard | Active |
| **Aider edit (legacy)** | Python-only edit-format compliance | Older |

⚠ **SWE-bench contamination**: OpenAI's 2025 audit found verbatim gold-patch leaks for frontier models on Verified. OpenAI moved to SWE-bench Pro. Treat 2025/2026 Verified scores >75% with skepticism.

⚠ **LiveCodeBench contamination**: DeepSeek base showed huge contamination delta on pre/post-Sep-2023 problems. Always cite a date slice (e.g., "LCB 2024-07 to 2024-11").

### Per-model scores

#### HumanEval / HumanEval+ / MBPP / MBPP+ (FP16/BF16, official)

| Model | HumanEval | HE+ | MBPP | MBPP+ | Source |
|---|---|---|---|---|---|
| Qwen2.5-Coder-7B-Instruct | 88.4 | 84.1 | 83.5 | 71.7 | Qwen2.5-Coder TR (arXiv 2409.12186) |
| Qwen2.5-Coder-14B-Instruct | 89.6 | 87.2 | 86.2 | 72.8 | Qwen2.5-Coder TR |
| Qwen2.5-Coder-32B-Instruct | 92.7 | 87.2 | 90.2 | 75.1 | Qwen2.5-Coder TR |
| Llama-3.1-8B-Instruct | 72.6 | — | — | 72.8 | Meta eval_details |
| Llama-3.3-70B-Instruct | 88.4 | — | — | 87.6 | Meta card |
| DeepSeek-Coder-V2-Lite-Instruct | 81.1 | — | — | 68.8 | DeepSeek-Coder-V2 paper |
| Mistral-Small-3.1 (24B) | 88.4 | — | 74.7 | — | Mistral self-report |
| Mistral-Small-3.2 (24B) | — | 88.99–92.90 | — | 78.33 | Mistral 3.2 update |
| Codestral-22B v0.1 | 86.6 | — | 91.2 | — | Mistral announcement |
| Phi-3-Medium-128k | 58.5 | — | 73.8 | — | Microsoft card |
| Phi-4 | 82.6 | — | — | — | Microsoft card |
| Granite-3.1-8B-Instruct | not published | — | not published | — | IBM card omits |
| Granite-3.3-8B-Instruct | 89.73 | 86.09 | — | — | IBM card |

#### LiveCodeBench Pass@1

| Model | LCB Pass@1 | Date Slice | Source |
|---|---|---|---|
| Qwen2.5-Coder-7B-Instruct | 18.2 | 2024-07 to 2024-11 | Qwen2.5-Coder TR |
| Qwen2.5-Coder-14B-Instruct | 23.4 | 2024-07 to 2024-11 | Qwen2.5-Coder TR |
| Qwen2.5-Coder-32B-Instruct | 31.4 | 2024-07 to 2024-11 | Qwen2.5-Coder TR |
| DeepSeek-Coder-V2-Lite-Instruct | 24.3 | 2024-Q3 | DeepSeek-Coder-V2 paper |
| Phi-4 | reported in TR (no exact extracted) | 2024-08 to 2025-01 | Microsoft Phi-4 TR |
| Codestral-22B v0.1 | community only; flagged contamination | — | — |
| All other 13 candidates | not officially reported | — | — |

#### Aider polyglot (Exercism, current standard)

| Model | % Correct | Edit Format |
|---|---|---|
| Qwen2.5-Coder-32B-Instruct | 16.4 (whole) / 8.0 (diff) | both tested |
| Codestral-25.01 | 11.1 | whole |

(No other 13-list models on the polyglot board.)

#### Aider edit (legacy Python)

| Model | Pass@1 | Pass@2 | Format |
|---|---|---|---|
| Qwen2.5-Coder-32B (Ollama Q8) | — | 72.9 | whole |
| Qwen2.5-Coder-14B | — | 69.2 | whole |
| Llama-3.1-8B-Instruct | 26.3 | 37.6 | — |
| codestral:22b-v0.1-q8_0 (Ollama) | 35.3 | 48.1 | whole |
| codestral-2405 (API) | 35.3 | 51.1 | whole |
| DeepSeek-Coder-V2 0724 | 57.9 | 72.9 | diff |

#### SWE-bench Verified

**None of the 13 wiki candidate models have published SWE-bench Verified scores.** Top of leaderboard May 2026: Claude Mythos 93.9, Opus 4.7 87.6, DeepSeek-V4-Pro-Max 80.6, Kimi K2.6 80.2, GLM-5 77.8, Qwen3.6-27B 77.2. Smallest open model in top 50: Qwen3.5-35B-A3B at 69.2. Sub-30B dense models typically score <15% with stock Aider/SWE-agent scaffolds.

### Important caveats

1. **All scores FP16/BF16.** No public AWQ INT4 SWE-bench / LCB / EvalPlus evaluations.
2. **Aider Ollama entries** are the only quantized public numbers — Q8 GGUF for Qwen2.5-Coder-32B, codestral 22b.
3. **Mistral-Small versioning** — 3.0 (2501) → 3.1 (2503) → 3.2 (2506). Pin model page to a specific version.
4. **Codestral vs Codestral-25.01** — Aider polyglot has the newer "25.01" (closed-weight); the open-weight 22B-v0.1 is on the legacy edit board.
5. **JS-rendered leaderboards** — primary pages were not directly extractable; cross-checks done via mirrors (llm-stats.com, pricepertoken.com, benchlm.ai) and primary papers.

## Pages updated on ingest
- [[concepts/benchmarks]]
- [[concepts/code-generation]]
- All [[wiki/comparisons/tool-calling-models-on-a10g|model pages]]
