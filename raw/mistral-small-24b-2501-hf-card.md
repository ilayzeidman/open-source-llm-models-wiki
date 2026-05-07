# Mistral-Small-24B-Instruct-2501 (Hugging Face model card)

source_url: https://huggingface.co/mistralai/Mistral-Small-24B-Instruct-2501
fetched: 2026-05-07

## Specifications
- Parameters: 24B
- Context: 32k tokens (per HF card)
- License: Apache 2.0
- Release date: January 2025
- Tokenizer: Tekken (131k vocabulary)
- Base: `mistralai/Mistral-Small-24B-Base-2501`

## Key features
- Native function calling and JSON output ("agentic" framing)
- Strong system-prompt adherence
- Multilingual: English, French, German, Spanish, Italian, Chinese, Japanese, Korean, Portuguese, Dutch, Polish

## Benchmarks (from card)
- MMLU Pro (5-shot CoT): 0.663
- GPQA Main (5-shot CoT): 0.453
- HumanEval (Instruct, pass@1): 0.848
- MATH (Instruct): 0.706
- MTBench Dev: 8.35
- WildBench: 52.27
- Arena Hard: 0.873
- IFEval: 0.829

## V7-Tekken instruct template
```
<s>[SYSTEM_PROMPT]<system prompt>[/SYSTEM_PROMPT][INST]<user message>[/INST]<assistant response></s>[INST]<user message>[/INST]
```

## vLLM serving (recommended)
```bash
pip install --upgrade vllm
pip install --upgrade mistral_common>=1.5.2

vllm serve mistralai/Mistral-Small-24B-Instruct-2501 \
  --tokenizer_mode mistral \
  --config_format mistral \
  --load_format mistral \
  --tool-call-parser mistral \
  --enable-auto-tool-choice
```
GPU: ~55 GB VRAM bf16/fp16

## Recommended sampling
- Temperature: 0.15

## Quant checkpoints (Ollama referenced from card)
- Q4_K_M: `ollama run mistral-small:24b-instruct-2501-q4_K_M`
- Q8_0: `ollama run mistral-small:24b-instruct-2501-q8_0`
- FP16: `ollama run mistral-small:24b-instruct-2501-fp16`
