# Mistral-Small-3.2-24B-Instruct-2506 (Hugging Face model card)

source_url: https://huggingface.co/mistralai/Mistral-Small-3.2-24B-Instruct-2506
fetched: 2026-05-07

## Specifications
- Parameters: 24B
- Context: ~128k (code references MAX_TOK=131072; same family as 3.1)
- License: Apache 2.0
- Release: June 2025 (2506)
- Base: `Mistral-Small-3.1-24B-Base-2503` (same base as 3.1)

## What's new vs 3.1
1. Better instruction-following
2. 2x reduction in infinite/repetition generations (1.29% vs 2.11%)
3. **More robust function calling template / tool-call-parser**

## Benchmark deltas (3.2 vs 3.1)
| Benchmark | 3.1 | 3.2 | Δ |
|---|---|---|---|
| Wildbench v2 | 55.6 | 65.33 | +9.73 |
| Arena Hard v2 | 19.56 | 43.10 | +23.54 |
| IF (internal) | 82.75 | 84.78 | +2.03 |
| MMLU | 80.62 | 80.50 | -0.12 |
| MMLU Pro (5-shot CoT) | 66.76 | 69.06 | +2.30 |
| MATH | 69.30 | 69.42 | +0.12 |
| GPQA Diamond (5-shot CoT) | 45.96 | 46.13 | +0.17 |
| MBPP Plus pass@5 | 74.63 | 78.33 | +3.70 |
| HumanEval Plus pass@5 | 88.99 | 92.90 | +3.91 |
| Infinite generations (lower=better) | 2.11 | 1.29 | -0.82 |

Vision basically unchanged.

## vLLM serving
```bash
vllm serve mistralai/Mistral-Small-3.2-24B-Instruct-2506 \
  --tokenizer_mode mistral --config_format mistral \
  --load_format mistral --tool-call-parser mistral \
  --enable-auto-tool-choice --limit-mm-per-prompt '{"image":10}' \
  --tensor-parallel-size 2
```
GPU: ~55 GB VRAM bf16/fp16

## Quirks
- Card explicitly highlights "excellent at function / tool calling tasks via vLLM"
- Use SYSTEM_PROMPT.txt template
- Temperature 0.15

## Quant checkpoints (community, referenced)
- bartowski/mistralai_Mistral-Small-3.2-24B-Instruct-2506-GGUF
- lmstudio-community/Mistral-Small-3.2-24B-Instruct-2506-GGUF
- unsloth/Mistral-Small-3.2-24B-Instruct-2506-GGUF
- unsloth/Mistral-Small-3.2-24B-Instruct-2506-unsloth-bnb-4bit
- RedHatAI/Mistral-Small-3.2-24B-Instruct-2506-NVFP4
