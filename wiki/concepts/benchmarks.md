---
tags: [concepts, benchmarks]
last_updated: 2026-05-07
source_count: 2
---

# Benchmarks

Reference page for what each benchmark measures and where to find current scores. Used by all [[index|model pages]] for citation consistency.

## Tool calling

| Benchmark | Measures | Notes | Source |
|---|---|---|---|
| **BFCL** (Berkeley Function-Calling Leaderboard) | Single/multi/parallel tool calls, irrelevance detection, multi-turn (V3) | Hosted by Gorilla. V1 (single), V2 (live data, parallel), V3 (multi-turn agentic, hallucination/irrelevance), V4 (agentic web/memory/format-sensitivity). **The standard.** | [[sources/bfcl-leaderboard-2026-05]] |
| **τ-bench** (tau-bench) | Multi-turn tool use in customer-service-style tasks | Closer to real agentic workloads. Llama-xLAM-2-70b: 56.2% pass@1; GPT-4o: 52.9%; Claude 3.5 Sonnet: 60.1%. | [[sources/xlam-family-cards]] |
| **API-Bank** | Tool selection from large API catalogs | Less common but useful for catalog-scale stress tests | — |
| **Gorilla benchmarks** | Original tool-selection paper | Largely subsumed by BFCL | — |

V1 saturated. **V3 multi-turn is the current discriminator.**

## Code generation

| Benchmark | Measures | Saturation | Source |
|---|---|---|---|
| **HumanEval / HumanEval+** | 164 Python problems; EvalPlus adds aggressive tests | Saturated >92% for top models | [[sources/code-benchmarks-2026-05]] |
| **MBPP / MBPP+** | ~378 sanitized Python problems (easier than HumanEval) | Same | [[sources/code-benchmarks-2026-05]] |
| **LiveCodeBench** | Algorithmic problems; date-stamped, contamination-resistant | Active, primary differentiator | [[sources/code-benchmarks-2026-05]] |
| **CodeContests** | Competitive programming (Codeforces-style) | Hard for sub-32B models | — |
| **SWE-bench Verified** | 500 human-validated GitHub issues; agent-scored on FAIL_TO_PASS + PASS_TO_PASS | Active; ⚠ contamination flag | [[sources/code-benchmarks-2026-05]] |
| **SWE-bench Lite / Pro / Multilingual / Multimodal** | Easier subset / post-contamination redo / language coverage / images | Active | [[sources/code-benchmarks-2026-05]] |
| **RepoBench** | Repo-level code completion | Tests cross-file context | — |
| **Aider polyglot** | 225 Exercism, 6 languages | **Current Aider standard** | [[sources/code-benchmarks-2026-05]] |
| **Aider edit (legacy)** | Python-only edit-format compliance | Older | [[sources/code-benchmarks-2026-05]] |

## General reasoning (for context)

- **MMLU / MMLU-Pro** — knowledge breadth, often quoted but weak signal for tool/code tasks.
- **GSM8K / MATH** — math reasoning.
- **BBH (BIG-Bench Hard)** — mixed reasoning suite.
- **IFEval** — instruction-following compliance; correlates with tool-calling reliability.
- **GPQA Diamond** — graduate-level science questions.
- **Arena-Hard / AlpacaEval-2.0** — preference-based, against frontier reference.

## Important caveats

> ⚠ **All public BFCL / SWE-bench / LCB / EvalPlus scores are FP16/BF16.** No public AWQ INT4 evaluations on these benchmarks. Aider's Ollama Q8 entries are the only quantized public tool/code numbers extant (Qwen2.5-Coder-32B Q8: 72.9 pass@2; Codestral 22B Q8: 48.1 pass@2). Local AWQ INT4 evaluations would need to be re-run. [Source: [[sources/code-benchmarks-2026-05]]]

> ⚠ **SWE-bench contamination** (OpenAI 2025 audit): verbatim gold-patch leaks found for frontier models on Verified. OpenAI moved to SWE-bench Pro. Treat 2025/2026 Verified scores >75% with skepticism.

> ⚠ **LiveCodeBench contamination**: DeepSeek base showed huge contamination delta on pre/post-Sep-2023 problems. Always cite a date slice (e.g., "LCB 2024-07 to 2024-11").

> ⚠ **BFCL version drift**: Llama 3.x scores in this wiki are V2; xLAM-7b-fc-r is V1; current public leaderboard is V3/V4. **Not directly comparable** across versions.

> ⚠ **Hermes-3 "BFCL ~91%" claim**: that figure is the well-formed-JSON rate, *not* BFCL overall accuracy. Don't quote as comparable.

> ⚠ **Qwen2.5-Coder absent from BFCL** despite strong tool-calling claims in its tech report.

## How to read scores in this wiki

- Numbers are presented as `<benchmark>: <score>`. Where applicable, also include `(<sub-task>)` and `(quant=<format>)` when known.
- All numbers in this wiki without an explicit `quant=` annotation are FP16/BF16.
- Always specify whether the score is for the **base** or **instruct** variant.
- Mark numbers `*(unverified — needs source)*` until backed by an ingested source.

## Related
- [[concepts/tool-selection]]
- [[concepts/code-generation]]
- [[wiki/comparisons/tool-calling-models-on-a10g]]

## Sources
- [[sources/bfcl-leaderboard-2026-05]]
- [[sources/code-benchmarks-2026-05]]
