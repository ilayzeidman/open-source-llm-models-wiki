# Mistral-Small-3.1-24B-Instruct-2503 (Hugging Face model card)

source_url: https://huggingface.co/mistralai/Mistral-Small-3.1-24B-Instruct-2503
fetched: 2026-05-07

## Specifications
- Parameters: 24B
- Context: 128k tokens
- License: Apache 2.0
- Release date: March 2025 (2503)
- Tokenizer: Tekken (131k vocab)
- Base: `Mistral-Small-3.1-24B-Base-2503`
- Multimodal: yes (image input)

## Benchmarks (text)
- MMLU: 80.62%
- MMLU Pro (5-shot CoT): 66.76%
- HumanEval: 88.41%
- MBPP: 74.71%
- MATH: 69.30%
- GPQA Main (5-shot CoT): 44.42%
- GPQA Diamond (5-shot CoT): 45.96%
- SimpleQA TotalAcc: 10.43%

## Vision benchmarks
- MMMU: 64.00%; MMMU PRO: 49.25%; Mathvista: 68.91%; ChartQA: 86.24%; DocVQA: 94.08%; AI2D: 93.72%; MM MT-Bench: 7.3

## Long context
- LongBench v2: 37.18%
- RULER 32K: 93.96%; RULER 128K: 81.20%

## vLLM serving
```bash
vllm serve mistralai/Mistral-Small-3.1-24B-Instruct-2503 \
  --tokenizer_mode mistral --config_format mistral --load_format mistral \
  --tool-call-parser mistral --enable-auto-tool-choice \
  --limit_mm_per_prompt 'image=10' --tensor-parallel-size 2
```
GPU: ~55 GB VRAM bf16/fp16

## Sampling
- Temperature: 0.15 recommended
- System prompt strongly recommended (SYSTEM_PROMPT.txt provided in repo)
- Note: card says "Transformers implementation is untested; vLLM recommended for production use"
