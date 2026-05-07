---
tags: [concepts, benchmarks]
last_updated: 2026-05-07
source_count: 0
---

# Benchmarks

Reference page for what each benchmark measures and where to find current scores. Used by all [[index|model pages]] for citation consistency.

## Tool calling

| Benchmark | Measures | Notes |
|---|---|---|
| **BFCL** (Berkeley Function-Calling Leaderboard) | Single/multi/parallel tool calls, irrelevance detection, multi-turn (v3) | The standard. Hosted by Gorilla. v1 (single), v2 (live data, parallel), v3 (multi-turn agentic). |
| **API-Bank** | Tool selection from large API catalogs | Less common but useful for catalog-scale stress tests |
| **NexusRaven evals** | Tool use across nested calls | Niche |
| **τ-bench (tau-bench)** | Multi-turn tool use in customer-service-style tasks | Closer to real agentic workloads |
| **Gorilla benchmarks** | Original tool-selection paper benchmarks | Largely subsumed by BFCL |

## Code generation

| Benchmark | Measures | Saturation status |
|---|---|---|
| **HumanEval / HumanEval+** | Single Python function from docstring | Mostly saturated for top open-source models; floor not ceiling |
| **MBPP / MBPP+** | Basic Python problems | Same as HumanEval |
| **LiveCodeBench** | Algorithmic problems, contamination-resistant (uses recent contest problems) | Active, good differentiator |
| **CodeContests** | Competitive programming (Codeforces-style) | Hard for sub-32B models |
| **SWE-bench Verified** | Real GitHub issues, 500 human-validated tasks | The headline "real engineering" benchmark. Open-source typically lags closed by a lot. |
| **SWE-bench Lite** | Easier subset of SWE-bench | More accessible |
| **RepoBench** | Repo-level code completion | Tests cross-file context |
| **CrossCodeEval** | Multi-language, cross-file completion | |
| **Aider's leaderboard** | Practical edit-style tasks via Aider | Tracks day-to-day usability |

## General reasoning (for context)

- **MMLU / MMLU-Pro** — knowledge breadth, often quoted but weak signal for tool/code tasks.
- **GSM8K / MATH** — math reasoning.
- **BBH (BIG-Bench Hard)** — mixed reasoning suite.

## How to read scores in this wiki

- Numbers are presented as `<benchmark>: <score>`. Where available, also include `(<sub-task>)` and `(quant=<format>)`.
- Mark numbers `*(unverified — needs source)*` until backed by an ingested source.
- Always specify whether the score is for the **base** or **instruct** variant, and whether it was measured at FP16 or quantized.

## Related
- [[concepts/tool-selection]]
- [[concepts/code-generation]]
- [[wiki/comparisons/tool-calling-models-on-a10g]]

## Sources
- (none yet)

## TODO / verify
- BFCL leaderboard methodology page
- SWE-bench Verified paper + leaderboard
- LiveCodeBench paper + leaderboard
- Aider's benchmark methodology
