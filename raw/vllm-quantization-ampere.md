# vLLM Quantization Support on NVIDIA Ampere (A10/A10G/A100)

source_url: https://docs.vllm.ai/en/latest/features/quantization/index.html
fetched: 2026-05-07

## Compatibility on Ampere (sm_80 / sm_86)

| Method                          | Supported on Ampere? |
|---------------------------------|----------------------|
| AWQ                             | ✅                   |
| GPTQ                            | ✅                   |
| Marlin (GPTQ / AWQ / FP8 / FP4) | ✅                   |
| INT8 (W8A8)                     | ✅                   |
| FP8 (W8A8)                      | ❌ (Ada+ / Hopper+)  |
| BitsAndBytes                    | ✅                   |
| GGUF                            | ✅                   |

## Key facts

- **Marlin INT4 kernels work on Ampere** — this is the recommended fast path for AWQ/GPTQ INT4 on A10G.
- **No native FP8 on Ampere.** FP8 tensor cores were introduced on Ada (sm_89) and Hopper (sm_90). On Ampere, FP8 in vLLM is unsupported per the matrix; some software paths can emulate FP8 via casting but get no hardware speedup.
- Footnote in vLLM docs: "Turing does not support Marlin MXFP4" — Ampere is unaffected.

## Practical guidance for A10G (24 GB)

For models that don't fit in BF16/FP16 weights:
- Use AWQ INT4 or GPTQ INT4 with the Marlin kernel for best throughput.
- INT8 (W8A8) is supported but typically used for sub-FP16 weight compression with FP16 activations; less common on a 24 GB box.
- Skip FP8 entirely — not supported on A10G.

## Open issues

- vLLM docs page is the latest snapshot; verify status when major vLLM release notes change quantization support.
