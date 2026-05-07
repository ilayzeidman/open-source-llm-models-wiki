# LiveCodeBench Methodology

source_url: https://livecodebench.github.io/
fetched: 2026-05-07

## Goal

"Holistic and contamination-free evaluation of LLMs for code." Avoid the well-documented HumanEval/MBPP saturation and contamination by sourcing fresh problems and dating each one.

## Problem source

Periodic competitive programming contests on:

- LeetCode
- AtCoder
- Codeforces

## Annotation

Each problem is tagged with its release date. Evaluation can then filter to problems released **after** a given model's training cutoff for an OOD comparison.

## Scenarios (4)

1. **Code Generation** (the headline metric in most reports): given problem statement, produce code that passes hidden tests.
2. **Self-Repair**: given failing code + error, produce corrected code.
3. **Test Output Prediction**: given function + input, predict output.
4. **Code Execution**: given code + input, predict program behavior.

Most papers cite "LiveCodeBench Pass@1" — that's scenario 1 unless otherwise noted.

## Versions / date slices

LiveCodeBench is "live" — problems accrue over time. Reports cite a date range:

- Original release: May 2023 – Feb 2024 (~300 problems).
- Common slices in 2024 reports: 2024-05 to 2024-08; 2024-07 to 2024-11 (Qwen2.5-Coder TR); 2024-08 to 2025-01 (Phi-4 TR).
- Internal "v1"…"v6" labels exist in the leaderboard UI corresponding to monthly cumulative releases.

## Metric

Pass@1 with greedy decoding (temperature 0, single sample). Some papers report Pass@1 averaged over 10 samples at temp 0.2.

## Contamination evidence

The original paper showed:
- DeepSeek (base) scores drop sharply on LeetCode problems posted after Sep 2023 → strong contamination signal.
- GPT-4 scores are stable across time → less contaminated.

This is why model authors increasingly report a specific recent slice (post-cutoff) rather than the full LCB.

## Submission

Researchers submit scores via PR to the LiveCodeBench GitHub. Public leaderboard updated on review.

## Caveats for our wiki

- **Quantization**: not part of the leaderboard. All scores assumed FP16/BF16 from authors.
- **Difficulty splits**: leaderboard breaks down Easy/Medium/Hard; total Pass@1 is the headline.
- **Open vs. agentic**: LCB is straight code-generation, not agentic. SWE-bench is the agentic counterpart.
