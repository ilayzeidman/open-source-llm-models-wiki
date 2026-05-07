# LiveCodeBench (May 2026)

source_url: https://livecodebench.github.io/leaderboard.html
fetched: 2026-05-07

## Status

The official LiveCodeBench leaderboard at https://livecodebench.github.io/leaderboard.html is JavaScript-rendered ("Loading leaderboard data..." in raw HTML).

## Methodology summary

source_url: https://livecodebench.github.io/

- **Problem source**: periodic contests on LeetCode, AtCoder, Codeforces.
- **Annotation**: each problem labeled with its release date so models can be evaluated only on problems released after their training cutoff.
- **Scenarios** (4): code generation, self-repair, test output prediction, code execution.
- **Metric**: Pass@1.
- **Versions**: data is sliced by date range (often referred to as v1–v6 in papers), e.g. "LCB 2024-05 to 2024-08" or "LCB 2024-08 to 2025-01". The Qwen2.5-Coder paper used "LiveCodeBench (2024-07∼2024-11)".
- **Contamination evidence**: DeepSeek base models showed sharp drops on post-2023-09 problems; GPT-4 stable.

## Per-model LiveCodeBench scores from primary sources

| Model | LCB Pass@1 | Date range | Source |
|-------|-----------|-----------|--------|
| Qwen2.5-Coder-7B-Instruct | 18.2 | 2024-07 to 2024-11 | Qwen2.5-Coder TR (arxiv 2409.12186) |
| Qwen2.5-Coder-14B-Instruct | 23.4 | 2024-07 to 2024-11 | Qwen2.5-Coder TR |
| Qwen2.5-Coder-32B-Instruct | 31.4 | 2024-07 to 2024-11 | Qwen2.5-Coder TR |
| DeepSeek-Coder-V2-Lite-Instruct | 24.3 | reported 2024-Q3 | DeepSeek-Coder-V2 paper |
| DeepSeek-Coder-V2 (236B) | 43.4 | reported 2024-Q3 | DeepSeek-Coder-V2 paper |
| Phi-4 | reported on LCB 2024-08 to 2025-01 (no specific number in fetched extracts) | 2024-08 to 2025-01 | Phi-4 tech report |
| Llama-3.1-8B-Instruct | not officially reported | — | — |
| Llama-3.3-70B-Instruct | not officially reported | — | — |
| Mistral-Small-3 / 3.1 | not officially reported | — | — |
| Codestral-22B v0.1 | community reports exist; contamination flagged in HumanEval Pro paper (aclanthology 2025.findings-acl.686) | — | — |
| Granite-3.1-8B / 4.1-8B | not on LCB | — | — |
| Hermes-3 | not on LCB | — | — |
| xLAM-7B / xLAM-2 | not on LCB (tool-calling model family) | — | — |
| Functionary-small | not on LCB | — | — |

## Notes on quantization

LCB scores are reported at FP16 / BF16 from model authors. Quantized scores (AWQ INT4, GPTQ) are not part of the official leaderboard and would require local re-evaluation.
