# vLLM Quantization Documentation

source_url: https://docs.vllm.ai/en/latest/features/quantization/index.html (and /fp8.html, /auto_awq.html)
fetched: 2026-05-07

## Supported Quantization Formats (vLLM 0.20.x, May 2026)

- AutoAWQ (deprecated upstream — recommended path now: llm-compressor's AWQ pipeline)
- GPTQModel
- BitsAndBytes
- GGUF
- INT4 W4A16
- INT8 W8A8
- FP8 W8A8
- NVIDIA Model Optimizer
- AMD Quark
- Intel Neural Compressor
- TorchAO
- Online Quantization (new unified frontend in 0.20)
- Quantized KV Cache
- FP8 ViT Encoder Attention

## Ampere (SM 8.0/8.6) GPU Support — A10G is sm_86

| Method | Ampere/A10G Support | Kernel | Notes |
|---|---|---|---|
| AWQ (W4A16) | yes | Marlin | preferred quant for A10G |
| GPTQ (W4A16) | yes | Marlin | well-tuned on Ampere |
| Marlin (covers GPTQ/AWQ/FP8/FP4) | yes (Turing+) | Marlin | "Turing does not support Marlin MXFP4" |
| INT8 W8A8 | yes | cutlass | |
| FP8 W8A8 | NO | — | requires compute capability >= 8.9 (Ada Lovelace / Hopper) |
| FP8 weight-only (W8A16, FP8 Marlin) | yes | Marlin | "FP8 models will run on compute capability >= 7.5 (Turing) as weight-only W8A16, utilizing FP8 Marlin." |
| BitsAndBytes | yes | bnb | |
| DeepSpeedFP | yes | | |
| GGUF | yes | | |

## FP8 W8A8 — exact wording from docs

"Only Hopper and Ada Lovelace GPUs are officially supported for W8A8."
"FP8 computation is supported on NVIDIA GPUs with compute capability >= 8.9 (Ada Lovelace, Hopper)."
"FP8 models will run on compute capability >= 7.5 (Turing) as weight-only W8A16, utilizing FP8 Marlin."

Implication for **A10G**: An "FP8" model checkpoint (e.g. Qwen3-FP8) **does load** on A10G but vLLM falls back to W8A16 weight-only execution via the FP8 Marlin kernel — activations are upcast. You do not get the H100/L40S compute speedups; you do get the ~50% weight-memory saving.

## AWQ on Ampere
- AutoAWQ library is upstream-deprecated; use llm-compressor's AWQ recipe to produce checkpoints (https://github.com/vllm-project/llm-compressor/tree/main/examples/awq).
- Serving: `--quantization awq` (or autodetected from `quantization_config` in `config.json`); `--quantization awq_marlin` to force the Marlin kernel.
- Best balance on A10G for 14B-30B class models.

## v0.20 quantization changes
- "Online quantization frontend" — unified path; `experts_int8` consolidated into FP8 online.
- `Sparse24` removed.
- `Petit NVFP4` quantization removed.
- TurboQuant 2-bit KV cache (4× capacity).
- Per-token-head INT8/FP8 KV cache quantization.
- W4A16 Autoround support on CPU/XPU.
- GPTQMarlin MoE reworked with new kernel infrastructure.
