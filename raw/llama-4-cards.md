# Llama 4 family — HF cards + Meta announcement

source_urls:
  - https://huggingface.co/meta-llama/Llama-4-Scout-17B-16E-Instruct
  - https://huggingface.co/meta-llama/Llama-4-Maverick-17B-128E-Instruct
  - https://www.llama.com/models/llama-4/
  - https://huggingface.co/blog/llama4-release
fetched: 2026-05-07

## Llama-4-Scout (17B-16E-Instruct)
- 109B total / **17B active** (MoE, 16 experts)
- License: Llama 4 Community License (commercial use under 700M MAU)
- Context: **10M** tokens (Meta's claim)
- Multimodal: text + image input → text + code output
- Training: ~40T tokens, 5M H100 GPU-hours
- Quant options: BF16 native, INT4 on-the-fly
- Hardware: "fits on a single H100 80 GB with int4"; AWS smallest = p5.48xlarge slice
- vLLM: standard `llama` parser path; tensor-parallel 1–2 recommended
- Footprint: BF16 ~218 GB → multi-GPU. INT4 ~55 GB → single H100 / L40S 8×

## Llama-4-Maverick (17B-128E-Instruct)
- ~400B total / 17B active (128 experts)
- License: Llama 4 Community
- Context: 1M tokens
- Multimodal

## Benchmarks (instruct)
| Benchmark | Scout | Maverick |
|---|---:|---:|
| MMLU Pro | 74.3 | 80.5 |
| GPQA Diamond | 57.2 | 69.8 |
| LiveCodeBench | 32.8 | 43.4 |
| MMMU | 73.4 | n/a |
| DocVQA | 94.4 | 94.4 |

Community testing: Maverick generally trails DeepSeek-V3.1 and Qwen3-Coder-480B
on coding/agent benchmarks. Scout is the better single-box deployable from this family.

## Tool calling notes
- Both support tool calling per Meta model card with documented prompt format.
- Community reports: less consistent JSON adherence than DeepSeek/Qwen3 — recommend
  output validation/retry in production.
