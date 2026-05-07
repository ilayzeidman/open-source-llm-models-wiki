# Granite-3.1-8B-Instruct (Hugging Face model card)

source_url: https://huggingface.co/ibm-granite/granite-3.1-8b-instruct
fetched: 2026-05-07

## Specifications
- Parameters: 8.1B (dense)
- Context: 128k tokens
- License: Apache 2.0
- Release: December 18, 2024
- Architecture: 40 layers, embed=4096, 32 heads, 8 KV heads, MLP=12800, SwiGLU, RoPE
- Pretraining tokens: 12T
- Languages: English, German, Spanish, French, Japanese, Portuguese, Arabic, Czech, Italian, Korean, Dutch, Chinese (12)

## Benchmarks (Open LLM Leaderboard V1)
- ARC-Challenge: 62.62
- HellaSwag: 84.48
- MMLU: 65.34
- TruthfulQA: 66.23
- Winogrande: 75.37
- GSM8K: 73.84
- Average: 71.31

## Open LLM Leaderboard V2
- IFEval: 72.08
- BBH: 34.09
- MATH Lvl 5: 21.68
- GPQA: 8.28
- MUSR: 19.01
- MMLU-Pro: 28.19
- Average: 30.55

## Capabilities listed
- Function-calling tasks (explicitly listed)
- Summarization, classification, extraction, QA, RAG, code, multilingual dialog, long-context

## Tool calling
- Native function-calling support
- vLLM tool parser flag: `--tool-call-parser granite` (per vLLM docs for Granite 3.1-8b)
- IBM Granite 3.0 announcement claims best-in-class scores on Berkeley Function Calling Leaderboard for its weight class

## Quantizations
- 30 community quantized models (llama.cpp, LM Studio, Jan, Ollama)

## Successor
- Card explicitly notes: newer version is `ibm-granite/granite-3.3-8b-instruct`
