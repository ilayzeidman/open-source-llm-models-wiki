# Qwen2.5-Coder-14B-Instruct — Hugging Face model card

source_url: https://huggingface.co/Qwen/Qwen2.5-Coder-14B-Instruct
fetched: 2026-05-07

## Identity
- HF model ID: `Qwen/Qwen2.5-Coder-14B-Instruct`
- Family: Qwen2.5-Coder (sizes 0.5B, 1.5B, 3B, 7B, 14B, 32B)
- Base: `Qwen/Qwen2.5-14B`
- Pre-instruct: `Qwen/Qwen2.5-Coder-14B`
- Tech report: arXiv 2409.12186

## Architecture / parameters
- Total parameters: 14.7B
- Non-embedding: 13.1B
- Layers: 48
- Hidden: 5,120
- Attention: 40 Q / 8 KV (GQA)
- Intermediate size: 13,824
- Tensor type: BF16

## Context
- Native: 32,768 tokens; full 131,072 with YaRN factor 4.0
- vLLM caveat: static YaRN only

## License
- Apache-2.0

## Training
- 5.5T tokens (continued pretraining over Qwen2.5)

## Benchmark scores (Qwen2.5-Coder tech report arXiv 2409.12186v3)
- HumanEval: 89.6 / HumanEval+: 87.2
- MBPP: 86.2 / MBPP+: 72.8
- BigCodeBench Complete: 81.0 / Hard: 22.3 / Instruct: 48.4
- LiveCodeBench (2024.07–2024.09): 23.4 Pass@1
- MultiPL-E: Python 89.0, Java 79.7, C++ 85.1, C# 84.2, TS 86.8, JS 84.5, PHP 80.1, Bash 47.5
- CRUXEval: Input-CoT 69.5 / Output-CoT 79.5
- Aider: Pass@1 58.6 / Pass@2 69.2
- BFCL: not in report
- SWE-bench: not in report

## Recommended quantized checkpoints on HF
- `Qwen/Qwen2.5-Coder-14B-Instruct-AWQ`
- `Qwen/Qwen2.5-Coder-14B-Instruct-GPTQ-Int4`
- `Qwen/Qwen2.5-Coder-14B-Instruct-GPTQ-Int8`
- 94 community quantizations (HF model tree)

## Chat template
- Qwen2.5 ChatML; `apply_chat_template(messages, tokenize=False, add_generation_prompt=True)`

## vLLM serving
```bash
pip install vllm
vllm serve Qwen/Qwen2.5-Coder-14B-Instruct
```
- Tool-call parser caveat: same as 7B — Qwen2.5 template advertises Hermes-style tool use, but Qwen2.5-Coder variants emit unreliable formatting (raw / fenced JSON instead of `<tool_call>` tags) per upstream issues. Dedicated `<tools>` parser upstreaming in vLLM PR #32931.

## Successor status
- Qwen3-Coder series superseded Qwen2.5-Coder in Feb 2026. No Qwen3-Coder-14B exact equivalent confirmed; check HF for current sizing.

## Release date
- 2024-11-12 (Qwen2.5-Coder family blog announcement; tech report v3 same date)
