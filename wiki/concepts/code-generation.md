---
tags: [concepts, code]
last_updated: 2026-05-07
source_count: 0
---

# Code generation

"Complex code" can mean very different things — the right model and benchmark depends on the task type.

## Task taxonomy

| Tier | Description | Representative benchmarks |
|---|---|---|
| **Single-function** | Implement a function from a docstring; small, self-contained | HumanEval, HumanEval+, MBPP, MBPP+ |
| **Algorithmic / competitive** | Solve programming-contest-style problems | LiveCodeBench, CodeContests, USACO |
| **Multi-file / repo-scale** | Edit across files, follow project conventions | RepoBench, EvalPlus's repo subset, CrossCodeEval |
| **Bug fixing on real repos** | Read an issue, locate code, write a patch that passes hidden tests | SWE-bench (Verified, Lite, Full), SWE-bench Multimodal |
| **Agentic SWE** | Multi-turn: explore, edit, run tests, iterate | SWE-bench (with agent harness), Aider's leaderboard |

Tool-calling and agentic SWE are closely linked — solving SWE-bench-style tasks essentially requires good [[concepts/tool-selection]].

## What "complex code" tends to mean for this research

Given the deployment target (single A10G, [[infrastructure/nvidia-dynamo]] + [[infrastructure/vllm]]) and the focus on tool selection, the relevant code tasks are likely to be:
- Producing patches to existing files (multi-file edits).
- Following project conventions (using existing utilities rather than reinventing).
- Generating code that calls tools / APIs correctly.
- Long-context reasoning across a repo.

This argues for models with:
- Strong **agentic** code scores (SWE-bench, LiveCodeBench), not just HumanEval.
- Long usable context (≥ 32k effective).
- Good tool-calling on top of code skills.

The Qwen2.5-Coder family ([[models/qwen2.5-coder-14b]], [[models/qwen2.5-coder-32b]]) and [[models/deepseek-coder-v2-lite]] are the leading open-source candidates here. [[models/codestral-22b]] is similar tier.

## How to read code benchmarks

- **HumanEval / MBPP (and +)**: small, mostly saturated by mid-2024; useful as a sanity check, not a differentiator.
- **LiveCodeBench**: evolving benchmark, less contaminated; better signal for current capability.
- **SWE-bench Verified**: human-validated subset; the most-cited "real engineering" benchmark. Sub-32B open-source typically scores in the single digits to low teens; 32B+ frontier-class models clear 20–40%+. *(unverified — needs source)*
- **Aider's leaderboard**: practical edit-style tasks; closely tracks day-to-day code-assistant quality.

## Related
- [[concepts/tool-selection]]
- [[concepts/benchmarks]]
- [[models/qwen2.5-coder-14b]], [[models/qwen2.5-coder-32b]], [[models/deepseek-coder-v2-lite]], [[models/codestral-22b]]

## Sources
- (none yet)

## TODO / verify
- Current SWE-bench Verified leaderboard
- LiveCodeBench leaderboard for open-source models
- Aider's edit benchmark leaderboard
- Recent code-model papers (Qwen2.5-Coder tech report, DeepSeek-Coder-V2)
