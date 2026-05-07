# Qwen2.5-Coder Series: Powerful, Diverse, Practical (Qwen blog)

source_url: https://qwenlm.github.io/blog/qwen2.5-coder-family/
fetched: 2026-05-07

## Release
- Release date: 2024-11-12 (entire 0.5B–32B family announced together with Instruct variants and quantized checkpoints)

## License
- Apache-2.0 for all six sizes

## Context length
- 128K tokens (with YaRN scaling on top of native 32K)

## Headline claim
- Qwen2.5-Coder-32B-Instruct matches GPT-4o on EvalPlus, LiveCodeBench, BigCodeBench among open-source code models.

## Selected scores called out in the blog
- 32B-Instruct: Aider 73.7, McEval 65.9, MdEval 75.2
- LiveCodeBench: 32B-Instruct surpasses GPT-4 (post on later cut: ~51.2)

## Practical artifacts shipped at announcement
- For each size: Base, Instruct, AWQ, GPTQ-Int4, GPTQ-Int8 official checkpoints under the `Qwen/` namespace
