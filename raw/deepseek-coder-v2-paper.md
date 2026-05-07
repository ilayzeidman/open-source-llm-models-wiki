# DeepSeek-Coder-V2: Breaking the Barrier of Closed-Source Models in Code Intelligence

source_url: https://arxiv.org/abs/2406.11931 (HTML: https://arxiv.org/html/2406.11931v1)
fetched: 2026-05-07

## Paper metadata
- arXiv: 2406.11931
- Submitted: 2024-06-17
- Authors: DeepSeek-AI

## Model lineup
| Variant | Total | Active | Context |
|---------|-------|--------|---------|
| DeepSeek-Coder-V2 (full)  | 236B | 21B  | 128K |
| DeepSeek-Coder-V2-Lite    | 16B  | 2.4B | 128K |
Each ships Base and Instruct.

## Training
- Continued pretrain on top of DeepSeek-V2 with 6T additional tokens
- 338 programming languages (up from 86 in DeepSeek-Coder v1)
- Lite total tokens: 10.2T

## Benchmarks (Lite-Instruct, 16B/2.4B-active)
- HumanEval: 81.1
- MBPP+: 68.8
- LiveCodeBench: 24.3
- USACO: 6.5
- FIM (mean): 86.4
- SWE-bench: 0.0 (model too small for harness)
- Aider: 44.4
- BBH: 61.2 / MMLU: 60.1
- GSM8K: 86.4 / MATH: 61.8
- ARC-E 88.9 / ARC-C 77.4
- Arena-Hard: 38.1

## Benchmarks (Full V2-Instruct, 236B/21B-active) for context
- HumanEval: 90.2
- MBPP+: 76.2
- LiveCodeBench: 43.4
- "Performance comparable to GPT-4-Turbo on code-specific tasks"

## License (paper-level)
"Permissive license, allowing for both research and unrestricted commercial use" — see DeepSeek Model License attached to the GitHub repo (LICENSE-MODEL).
