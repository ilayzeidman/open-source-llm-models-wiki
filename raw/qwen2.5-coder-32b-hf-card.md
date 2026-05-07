# Qwen2.5-Coder-32B-Instruct — Hugging Face model card

source_url: https://huggingface.co/Qwen/Qwen2.5-Coder-32B-Instruct
fetched: 2026-05-07

## Identity
- HF model ID: `Qwen/Qwen2.5-Coder-32B-Instruct`
- Family: Qwen2.5-Coder
- Base: `Qwen/Qwen2.5-32B`
- Pre-instruct: `Qwen/Qwen2.5-Coder-32B`
- Tech report: arXiv 2409.12186

## Architecture / parameters
- Total: 32.5B (non-embedding 31.0B)
- Layers: 64
- Hidden: 5,120
- Attention: 40 Q / 8 KV (GQA)
- Intermediate: 27,648
- Tensor type: BF16

## Context
- Native 32,768 / extended 131,072 via YaRN factor 4.0

## License
- Apache-2.0

## Benchmark scores (tech report arXiv 2409.12186v3)
- HumanEval: 92.7 / HumanEval+: 87.2
- MBPP: 90.2 / MBPP+: 75.1
- BigCodeBench Complete: 83.0 / Hard: 26.4 / Instruct: 49.6
- LiveCodeBench (2024.07–2024.09): 31.4 Pass@1 (independent reports cite up to 51.2 on later cuts)
- MultiPL-E: Python 92.7, Java 80.4, C++ 79.5, C# 82.9, TS 86.8, JS 85.7, PHP 78.9, Bash 48.1
- CRUXEval: Input-CoT 75.2 / Output-CoT 83.4
- Aider: Pass@1 60.9 / Pass@2 73.7
- McEval: 65.9 (per Qwen blog)
- MdEval: 75.2 (per Qwen blog)
- BFCL: not in tech report
- SWE-bench: not in tech report

## Headline claim
> "Qwen2.5-Coder-32B-Instruct has become the current state-of-the-art open-source codeLLM, with its coding abilities matching those of GPT-4o."

## Recommended quantized checkpoints on HF
- `Qwen/Qwen2.5-Coder-32B-Instruct-AWQ` (official, INT4) — 1.13M downloads/month at fetch
- `Qwen/Qwen2.5-Coder-32B-Instruct-GPTQ-Int4`
- `Qwen/Qwen2.5-Coder-32B-Instruct-GPTQ-Int8`
- 119 community quantizations

## Chat template
- ChatML via `apply_chat_template`

## vLLM serving
```bash
pip install vllm
vllm serve Qwen/Qwen2.5-Coder-32B-Instruct       # BF16, will not fit on 24 GB
vllm serve Qwen/Qwen2.5-Coder-32B-Instruct-AWQ   # INT4, fits on single A10G with care
```
- Tool-call parser: same Hermes-style chat-template story as 7B/14B; Qwen2.5-Coder is known-flaky on `<tool_call>` emission; community workarounds exist.

## Successor status
- Qwen3-Coder family (Feb 2026) supersedes Qwen2.5-Coder. Qwen3-Coder-Next reports >70% on SWE-bench Verified vs Qwen2.5-Coder-32B's far lower mark.

## Release date
- 2024-11-12 (family blog announcement)
