# Qwen2.5-Coder-7B-Instruct — Hugging Face model card

source_url: https://huggingface.co/Qwen/Qwen2.5-Coder-7B-Instruct
fetched: 2026-05-07

## Identity
- HF model ID: `Qwen/Qwen2.5-Coder-7B-Instruct`
- Family: Qwen2.5-Coder (sizes 0.5B, 1.5B, 3B, 7B, 14B, 32B)
- Base: `Qwen/Qwen2.5-7B`
- Pre-instruct: `Qwen/Qwen2.5-Coder-7B`
- Tech report arXiv: 2409.12186 (submitted 2024-09-18, latest revision v3 2024-11-12)

## Architecture / parameters
- Total parameters: 7.61B
- Non-embedding parameters: 6.53B
- Layers: 28
- Attention heads: 28 Q / 4 KV (GQA)
- Type: Causal LM, RoPE, SwiGLU, RMSNorm, Attention QKV bias
- Tensor type: BF16

## Context
- Native: 32,768 tokens (config.json default)
- With YaRN scaling factor 4.0: 131,072 tokens
- vLLM caveat: only static YaRN supported — leaving YaRN on degrades short-context quality

## License
- Apache-2.0 (commercial use allowed, no restrictions)

## Training
- 5.5T tokens; source code, text-code grounding, synthetic data
- Two-stage: pretraining + post-training (instruct)

## Benchmark scores (per Qwen2.5-Coder tech report, arXiv 2409.12186v3, Table 16/17/18/19)
- HumanEval: 88.4 / HumanEval+: 84.1
- MBPP: 83.5 / MBPP+: 71.7
- BigCodeBench Complete: 76.9 / Hard: 16.2 / Instruct: 41.0
- LiveCodeBench (2024.07–2024.09): 18.2 Pass@1
- MultiPL-E: Python 87.8, Java 76.5, C++ 75.6, C# 80.3, TS 81.8, JS 83.2, PHP 78.3, Bash 48.7
- CRUXEval: Input-CoT 65.8 / Output-CoT 65.9
- Aider: Pass@1 55.6 / Pass@2 68.4
- BFCL: not evaluated in tech report
- SWE-bench: not evaluated in tech report

## Recommended quantized checkpoints on HF
- `Qwen/Qwen2.5-Coder-7B-Instruct-AWQ` (official AWQ INT4)
- `Qwen/Qwen2.5-Coder-7B-Instruct-GPTQ-Int4`
- `Qwen/Qwen2.5-Coder-7B-Instruct-GPTQ-Int8`
- 182 community quantizations indexed via HF model tree

## Chat template
- Standard Qwen2.5 ChatML template via `tokenizer.apply_chat_template`
- System prompt suggested: "You are Qwen, created by Alibaba Cloud. You are a helpful assistant."

## vLLM serving
```bash
pip install vllm
vllm serve Qwen/Qwen2.5-Coder-7B-Instruct
```
- Tool-call parser: model card says nothing. vLLM docs state Qwen2.5 chat template includes Hermes-style tool support, so `--tool-call-parser hermes --enable-auto-tool-choice` is the canonical flag — but multiple GitHub issues (#10952, #29192, #32926) report Qwen2.5-Coder specifically does NOT emit `<tool_call>` tags reliably and may emit raw JSON or fenced JSON instead. A dedicated `<tools>`-tag parser is being upstreamed (PR #32931).
- Requirement: `transformers >= 4.37.0` (else `KeyError: 'qwen2'`).

## Successor status
- Qwen3-Coder family released February 2026 (per Qwen blog). Qwen3-Coder-Next claims >70% SWE-bench Verified with SWE-Agent, far ahead of Qwen2.5-Coder.
- For A10G fit purposes Qwen2.5-Coder-7B-Instruct remains a credible target if Qwen3-Coder lacks an equivalent-sized open checkpoint at the chosen quant.
