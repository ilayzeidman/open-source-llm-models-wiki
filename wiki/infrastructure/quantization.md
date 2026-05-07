---
tags: [infrastructure, quantization]
last_updated: 2026-05-07
source_count: 0
---

# Quantization on A10G

The A10G has 24 GB and is **memory-bandwidth-bound** at decode (~600 GB/s). Quantization is therefore both a *fit* lever (more model in less VRAM) and a *throughput* lever (fewer bytes per token). See [[hardware/a10g-g5xlarge]].

## Backend tradeoffs *(unverified — needs source)*

| Format | Bits | Calibration needed? | Quality vs FP16 | A10G speed (vLLM) | Memory savings | Notes |
|---|---|---|---|---|---|---|
| **BF16 / FP16** | 16 | no | baseline | baseline | 1× | reference |
| **AWQ INT4** | 4 (W only) | yes (data-aware) | ≈ −0.5 to −1.5pt on most benchmarks | **fast** (Marlin kernels) | ~4× | best default on Ampere |
| **GPTQ INT4** | 4 (W only) | yes | ≈ AWQ, sometimes slightly worse | fast (Marlin kernels) | ~4× | older format, very widely available |
| **INT8 SmoothQuant** | 8 (W+A) | yes | ≈ −0.2pt | medium | ~2× | good middle ground |
| **INT8 weight-only** | 8 (W only) | no | ≈ −0.1pt | medium | ~2× | safest 2× compression |
| **FP8 (E4M3)** | 8 | no/light | ≈ FP16 | **limited on A10G** — Ampere has no FP8 tensor cores; emulated path only | ~2× | best on H100/H200 |
| **NF4 / BitsAndBytes** | 4 | no | similar to AWQ but variable | slower than AWQ for serving | ~4× | training-friendly, not the best serving choice |
| **GGUF (Q4_K_M / Q5_K_M / ...)** | 4–8 | no | similar to AWQ at same bits | **N/A in vLLM** — use llama.cpp | ~4× at Q4 | great for CPU/Apple Silicon, off-track for vLLM |

Headline rule for this hardware: **AWQ INT4 (or GPTQ INT4) is the default for any model that doesn't fit comfortably at FP16.** INT8 is the conservative choice when an extra ~1pt of quality matters and the model still fits.

## What fits on 24 GB at what quant

(See [[hardware/a10g-g5xlarge]] for the full table; reproduced verdicts here.)

| Model size | FP16 | INT8 | INT4 (AWQ/GPTQ) |
|---|---|---|---|
| 7–8B | ✅ | ✅ | ✅ (huge KV budget left) |
| 13–14B | ❌ | ✅ tight | ✅ |
| 22–24B | ❌ | ⚠ very tight | ✅ |
| 32B | ❌ | ❌ | ⚠ tight (small KV budget) |
| 70B | ❌ | ❌ | ❌ → multi-GPU |

## Tool-calling and code: does quantization hurt the right things?

Open question — and one of the more important ones for this research:

- General benchmarks (MMLU, HumanEval) typically lose 0.5–1.5 points at INT4. *(unverified)*
- **Tool-calling JSON compliance** is a structured-output task; it's plausible that INT4 hurts more here, but constrained decoding ([[infrastructure/vllm]] §Structured outputs) compensates well.
- **Long-context retrieval** (needle-in-haystack) sometimes degrades at aggressive quant — cited concern but not always confirmed.
- **Multi-step agentic tasks** (e.g. SWE-bench-style) — unclear how much INT4 hurts. Worth testing per-model.

For seeded model pages, default recommendation is AWQ INT4 unless the model fits at FP16 on a single A10G; flag any model where quant-aware benchmarks meaningfully diverge from FP16.

## Related
- [[hardware/a10g-g5xlarge]]
- [[infrastructure/vllm]]
- [[concepts/tool-selection]]
- [[concepts/code-generation]]

## Sources
- (none yet)

## TODO / verify
- AWQ paper, GPTQ paper, SmoothQuant paper — original quality results
- vLLM docs on Marlin kernel coverage and AWQ/GPTQ throughput on Ampere
- BFCL / HumanEval ablations at FP16 vs INT4 for major model families
- Hugging Face quantized checkpoints for each candidate model
