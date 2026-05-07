# GPT-OSS family — OpenAI open-weight (gpt-oss-20b / gpt-oss-120b)

source_urls:
  - https://huggingface.co/openai/gpt-oss-20b
  - https://huggingface.co/openai/gpt-oss-120b
  - https://openai.com/index/introducing-gpt-oss/
  - OpenAI model card PDF (Aug 5, 2025)
  - https://developers.openai.com/cookbook/articles/gpt-oss/run-vllm
fetched: 2026-05-07

## gpt-oss-20b
- 21B total / **3.6B active** (MoE)
- License: **Apache-2.0**
- Context: 128K
- Native format: **MXFP4** (post-training MoE-weight quantization)
- Disk: ~12–13 GB; runtime VRAM: 16 GB minimum
- Configurable reasoning effort: low / medium / high
- Uses **Harmony response format** (mandatory)

### Benchmarks
| Benchmark | Low | Med | High |
|---|---:|---:|---:|
| SWE-bench Verified | 37.4 | 53.2 | 60.7 |
| GPQA Diamond | 56.8 | 66.0 | — |
| GPQA Diamond + tools | — | 67.1 | 74.2 |

Strong on Codeforces and Tau-Bench tool-calling (claimed near o4-mini parity in
medium/high reasoning).

## gpt-oss-120b
- 117B total / **5.1B active** (MoE)
- License: Apache-2.0
- Context: 128K
- Native: MXFP4; runtime ~63 GB → "single 80 GB GPU" (H100/MI300X)
- Beats o3-mini on Codeforces, MMLU, HLE; matches o4-mini on tool-calling tasks

## Critical hardware note (MXFP4)
- **MXFP4 tensor cores require Hopper (sm_90) or newer** → **H100 / H200 / B200 / RTX 5090**
- A10G (Ampere sm_86), L4/L40S (Ada sm_89), A100 (sm_80) **do NOT have MXFP4 hardware**
- vLLM emulates MXFP4 on Ampere/Ada via dequant-to-BF16, **but the weights then take
  ~2× the disk size in VRAM** — gpt-oss-20b ≈ 24+ GB BF16 (no longer fits 16 GB),
  gpt-oss-120b BF16 dequant ≈ 234 GB
- **Practical**: for AWS, gpt-oss-* is best-served on p5/p5e/p6-b200, or on a single
  L40S **only** with custom AWQ INT4 re-quantization (community quants exist)

## vLLM serving
- Requires custom build: `uv pip install --pre vllm==0.10.1+gptoss --extra-index-url https://wheels.vllm.ai/gpt-oss/`
- `vllm serve openai/gpt-oss-20b` (single H100) — Harmony response format auto-applied
- Tool-call parser: `--tool-call-parser openai` (mainline since 0.10.1+gptoss)
