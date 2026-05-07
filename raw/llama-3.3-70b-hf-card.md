# Llama-3.3-70B-Instruct (Hugging Face model card)

source_url: https://huggingface.co/meta-llama/Llama-3.3-70B-Instruct
fetched: 2026-05-07

## Specifications
- Parameters: 70B (dense)
- Context: 128k tokens
- Release date: December 6, 2024
- Knowledge cutoff: December 2023
- License: Llama 3.3 Community License Agreement — https://github.com/meta-llama/llama-models/blob/main/models/llama3_3/LICENSE
- Same 700M MAU clause as Llama 3.1
- Languages: English, German, French, Italian, Portuguese, Hindi, Spanish, Thai
- Acceptable Use Policy: https://www.llama.com/llama3_3/use-policy
- Pretraining tokens: 15T+
- Compute: 7.0M H100-80GB GPU hours

## License attribution requirements
- Redistribution must include: copy of agreement, "Built with Llama" notice
- Derived models must include "Llama" at start of name
- Notice line: "Llama 3.3 is licensed under the Llama 3.3 Community License, Copyright © Meta Platforms, Inc. All Rights Reserved."

## Benchmarks (Instruction-tuned)

| Benchmark | Llama 3.1 70B | Llama 3.3 70B | Llama 3.1 405B |
|---|---|---|---|
| MMLU (CoT, 0-shot) | 86.0 | 86.0 | 88.6 |
| MMLU Pro (CoT, 5-shot) | 66.4 | 68.9 | 73.3 |
| IFEval | 87.5 | 92.1 | 88.6 |
| GPQA Diamond (CoT, 0-shot) | 48.0 | 50.5 | 49.0 |
| HumanEval (0-shot, pass@1) | 80.5 | 88.4 | 89.0 |
| MBPP EvalPlus base (0-shot) | 86.0 | 87.6 | 88.6 |
| MATH (CoT, 0-shot) | 68.0 | 77.0 | 73.8 |
| BFCL v2 (overall_ast_summary/macro_avg/valid) | 77.5 | 77.3 | 81.1 |
| MGSM | 86.9 | 91.1 | 91.6 |

## Tool calling
- Chat template supports `tools=[...]` arg with python callable schema
- Same transformers chat-template flow as 3.1
- Recommended deployment: vLLM `vllm serve "meta-llama/Llama-3.3-70B-Instruct"`
- Quantization: BitsAndBytes 8-bit / 4-bit examples in card

## Successor
- Llama 4 (Scout, Maverick, Behemoth) released April 2025 — MoE architecture with native multimodality. Llama 3.3 70B remains the most recent dense ~70B class checkpoint from Meta.
