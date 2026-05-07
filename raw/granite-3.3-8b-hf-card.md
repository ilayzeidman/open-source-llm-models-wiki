# Granite-3.3-8B-Instruct (Hugging Face model card)

source_url: https://huggingface.co/ibm-granite/granite-3.3-8b-instruct
fetched: 2026-05-07

## Specifications
- Parameters: 8B
- Context: 128k tokens
- License: Apache 2.0
- Release: April 16, 2025
- Languages: same 12 as Granite 3.1

## Benchmarks
- Arena-Hard: 57.56
- AlpacaEval-2.0: 62.68
- MMLU: 65.54
- HumanEval: 89.73
- HumanEval+: 86.09
- IFEval: 74.82
- GSM8K: 80.89
- MATH-500: 69.02
- AIME24: 8.12

## Reasoning/thinking mode
- Special tags `<think>...</think>` and `<response>...</response>`
- Enable via `tokenizer.apply_chat_template(conv, thinking=True, ...)`

## Tool calling
- Listed capability: function-calling tasks
- vLLM: `vllm serve ibm-granite/granite-3.3-8b-instruct`
- Tool parser flag: `--tool-call-parser granite` (per vLLM docs; same as 3.1)

## Quantizations
- 29 community quantized models for llama.cpp, LM Studio, Jan, Ollama
- Available at NVIDIA NIM: docs.api.nvidia.com/nim/reference/ibm-granite-3_3-8b-instruct

## Notes vs 3.1
- HumanEval up from undocumented to 89.73 (huge code improvement)
- IFEval from 72.08 → 74.82
- Adds <think> reasoning mode
