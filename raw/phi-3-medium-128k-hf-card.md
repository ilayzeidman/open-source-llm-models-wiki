# Phi-3-medium-128k-instruct (Hugging Face model card)

source_url: https://huggingface.co/microsoft/Phi-3-medium-128k-instruct
fetched: 2026-05-07

## Specifications
- Parameters: 14B (dense)
- Context: 128k
- License: MIT
- Release: May 21, 2024
- Knowledge cutoff: October 2023
- Pretraining: 4.8T tokens (10% multilingual)
- Tokenizer vocab: 32,064

## Benchmarks
- MMLU (5-shot): 76.6%
- HumanEval (0-shot): 58.5%
- MBPP (3-shot): 73.8%
- GSM8K (CoT, 8-shot): 87.5%
- BigBench Hard (3-shot): 77.9%

Category averages: Reasoning 83.2 / Code 64.2 / LangUnderstanding 75.3 / Math 52.9 / Multilingual 62.2

## Chat template
```
<|user|>
Question<|end|>
<|assistant|>
```
- Use `<s>` BOS at start
- `trust_remote_code=True` required when loading
- Flash attention required (A100/A6000/H100)

## Tool calling
- **Not natively supported** — model card does not document a tool-calling template or vLLM tool parser
- This is a key gap vs the other models in this set

## Quantizations
- ONNX: int4 DML, fp16 CUDA, int4 CUDA, int4 CPU/Mobile
- Many community GGUF / AWQ variants exist
