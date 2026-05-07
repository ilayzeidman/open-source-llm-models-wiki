# Phi-4 (Hugging Face model card)

source_url: https://huggingface.co/microsoft/phi-4
fetched: 2026-05-07

## Specifications
- Parameters: 14B (dense decoder-only)
- Context: 16k tokens (downgrade from Phi-3-medium-128k)
- License: MIT
- Release: December 12, 2024
- Knowledge cutoff: June 2024 (and earlier for public data)
- Training: 9.8T tokens, 21 days, 1920 H100-80G GPUs
- Primary language: English (~92%)

## Benchmarks
- MMLU: 84.8
- HumanEval: 82.6
- MATH: 80.4
- MGSM: 80.6
- GPQA: 56.1
- DROP: 75.5
- SimpleQA: 3.0  (very low — model card calls out factual accuracy as a limitation)

## Chat template
```
<|im_start|>system<|im_sep|>...<|im_end|>
<|im_start|>user<|im_sep|>...<|im_end|>
<|im_start|>assistant<|im_sep|>
```

## Tool calling
- **Not natively supported in base Phi-4 14B.** The model card does not mention function calling.
- Phi-4-mini (3.8B) released later DOES support function calling natively, but that is a different model.
- A community discussion (huggingface.co/microsoft/phi-4/discussions/7) confirms Phi-4 14B lacks native function-calling support.
- vLLM has no `phi4_*` tool parser as of mid-2025; Phi-4-mini support was tracked in vllm-project/vllm#14682.

## Successor relationship
- Phi-4 (14B, Dec 2024) supersedes Phi-3-medium (14B, May 2024) on most benchmarks (HumanEval 82.6 vs 58.5, MMLU 84.8 vs 76.6, MATH 80.4 vs 52.9)
- BUT context dropped from 128k → 16k
- AND tool calling is still absent — for an agentic workload, Phi-4 14B is not a drop-in upgrade for tool selection

## Quantizations
- 149 community quantized models (llama.cpp, LM Studio, Jan, Ollama)
