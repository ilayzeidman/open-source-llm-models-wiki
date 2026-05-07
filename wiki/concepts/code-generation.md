---
tags: [concepts, code]
last_updated: 2026-05-07
source_count: 1
---

# Code generation

"Complex code" can mean very different things — the right model and benchmark depends on the task type.

## Task taxonomy

| Tier | Description | Representative benchmarks |
|---|---|---|
| **Single-function** | Implement a function from a docstring; small, self-contained | HumanEval, HumanEval+, MBPP, MBPP+ |
| **Algorithmic / competitive** | Solve programming-contest-style problems | LiveCodeBench, CodeContests, USACO |
| **Multi-file / repo-scale** | Edit across files, follow project conventions | RepoBench, EvalPlus repo subset, CrossCodeEval |
| **Bug fixing on real repos** | Read an issue, locate code, write a patch that passes hidden tests | SWE-bench (Verified, Lite, Full), SWE-bench Pro |
| **Agentic SWE** | Multi-turn: explore, edit, run tests, iterate | SWE-bench (with agent harness), Aider's leaderboard |

Tool-calling and agentic SWE are closely linked — solving SWE-bench-style tasks essentially requires good [[concepts/tool-selection]].

## Open-source models at sub-30B-dense scale: where they stand (May 2026)

[Source: [[sources/code-benchmarks-2026-05]], [[sources/qwen2.5-coder-tech-report]]]

For HumanEval (the basic correctness floor):

| Model | HumanEval | HumanEval+ | License | Notes |
|---|---|---|---|---|
| Qwen2.5-Coder-32B-Instruct | 92.7 | 87.2 | Apache-2.0 | tightest fit on A10G AWQ |
| Qwen2.5-Coder-14B-Instruct | 89.6 | 87.2 | Apache-2.0 | A10G sweet spot |
| Granite-3.3-8B-Instruct | 89.73 | 86.09 | Apache-2.0 | enterprise generalist with code |
| Qwen2.5-Coder-7B-Instruct | 88.4 | 84.1 | Apache-2.0 | cheapest code 7B |
| Llama-3.3-70B-Instruct | 88.4 | — | Llama 3.3 community | multi-GPU only |
| Mistral-Small-3.2 (24B) | — | 92.90 (HE+ pass@5) | Apache-2.0 | A10G fits AWQ |
| Codestral-22B v0.1 | 86.6 | — | **MNPL — non-commercial** | A10G fits AWQ |
| Phi-4 (14B) | 82.6 | — | MIT | **no tool calling** — agentic disqualified |
| DeepSeek-Coder-V2-Lite | 81.1 | — | DeepSeek License | MoE 16B/2.4B active |
| Llama-3.1-8B-Instruct | 72.6 | — | Llama 3.1 community | weaker on code |
| Phi-3-Medium-128k | 58.5 | — | MIT | **no tool calling** |

For LiveCodeBench (better differentiator):

| Model | LCB | Date Slice |
|---|---|---|
| Qwen2.5-Coder-32B-Instruct | 31.4 | 2024-07 to 2024-11 |
| DeepSeek-Coder-V2-Lite | 24.3 | 2024-Q3 |
| Qwen2.5-Coder-14B-Instruct | 23.4 | 2024-07 to 2024-11 |
| Qwen2.5-Coder-7B-Instruct | 18.2 | 2024-07 to 2024-11 |

For Aider polyglot (current standard, agentic edit):

| Model | % Correct |
|---|---|
| Qwen2.5-Coder-32B-Instruct | 16.4 (whole) / 8.0 (diff) |
| (no other in-scope model on polyglot board) | — |

For SWE-bench Verified (real GitHub issues):

**None of the 13 wiki candidate models have published SWE-bench Verified scores.** Top of leaderboard May 2026: Claude Mythos 93.9, Opus 4.7 87.6, DeepSeek-V4-Pro-Max 80.6. Smallest open model in top 50: Qwen3.5-35B-A3B at 69.2. Sub-30B dense models typically score <15% on Verified.

> ⚠ **SWE-bench contamination**: OpenAI's 2025 audit found verbatim gold-patch leaks for frontier models on Verified. OpenAI moved to SWE-bench Pro. Treat 2025/2026 Verified scores >75% with skepticism.

## What "complex code" tends to mean for this research

Given the deployment target (single A10G, [[infrastructure/nvidia-dynamo]] + [[infrastructure/vllm]]) and the focus on tool selection, the relevant code tasks are:
- Producing patches to existing files (multi-file edits).
- Following project conventions (using existing utilities rather than reinventing).
- Generating code that calls tools / APIs correctly.
- Long-context reasoning across a repo.

This argues for models with:
- Strong **agentic** code scores (LiveCodeBench, Aider polyglot), not just HumanEval.
- Long usable context (≥ 32k effective).
- Good tool-calling on top of code skills — see [[concepts/tool-selection]].

The **Qwen2.5-Coder family** is the leading open-source candidate; **Qwen3-Coder** has shipped (Feb 2026) as the successor and claims significantly higher SWE-bench Verified scores. [[models/deepseek-coder-v2-lite]] is the MoE alternative. [[models/codestral-22b]] is similar tier but **MNPL-licensed (non-commercial)**.

## How to read code benchmarks

- **HumanEval / MBPP (and +)**: small, mostly saturated by mid-2024; useful as a sanity check, not a differentiator.
- **LiveCodeBench**: contamination-resistant; better signal for current capability. Always cite a date slice.
- **SWE-bench Verified**: human-validated subset (500 tasks); the most-cited "real engineering" benchmark.
- **Aider's leaderboard**: practical edit-style tasks; closely tracks day-to-day code-assistant quality.

## Related
- [[concepts/tool-selection]]
- [[concepts/benchmarks]]
- [[models/qwen2.5-coder-14b]], [[models/qwen2.5-coder-32b]], [[models/deepseek-coder-v2-lite]], [[models/codestral-22b]]

## Sources
- [[sources/code-benchmarks-2026-05]]
- [[sources/qwen2.5-coder-tech-report]]
