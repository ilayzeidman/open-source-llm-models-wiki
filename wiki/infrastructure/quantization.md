---
tags: [infrastructure, quantization]
last_updated: 2026-05-07
source_count: 2
---

# Quantization on A10G

The A10G has 24 GB and is **memory-bandwidth-bound** at decode (~600 GB/s). Quantization is therefore both a *fit* lever (more model in less VRAM) and a *throughput* lever (fewer bytes per token). See [[hardware/a10g-g5xlarge]].

## Backend tradeoffs (vLLM 0.20.x on Ampere sm_86)

[Source: [[sources/vllm-quantization-docs]]]

| Format | Bits | Calibration | Quality vs FP16 | A10G speed | Memory | Kernel | Notes |
|---|---|---|---|---|---|---|---|
| **BF16 / FP16** | 16 | no | baseline | baseline | 1× | — | reference |
| **AWQ INT4** | 4 (W only) | yes | ≈ −0.5 to −1.5pt | **fast** | ~4× | `awq_marlin` | **default for ≥14B on A10G** |
| **GPTQ INT4** | 4 (W only) | yes | ≈ AWQ | fast | ~4× | `gptq_marlin` | older format, equivalent on Ampere |
| **INT8 SmoothQuant** | 8 (W+A) | yes | ≈ −0.2pt | medium | ~2× | cutlass | conservative middle ground |
| **INT8 weight-only** | 8 (W only) | no | ≈ −0.1pt | medium | ~2× | — | safest 2× compression |
| **FP8 W8A8** | 8 | no/light | ≈ FP16 | **❌ on A10G** — Ampere lacks FP8 tensor cores | ~2× | — | needs sm ≥ 8.9 (Ada/Hopper+) |
| **FP8 W8A16** (weight-only fallback) | 8 wt / 16 act | no | ≈ FP16 | medium | ~2× weights | FP8 Marlin | A10G silently runs FP8 checkpoints in this mode — saves weight memory, no compute speedup |
| **NF4 / BitsAndBytes** | 4 | no | similar to AWQ but variable | ⚠ slower than AWQ for serving | ~4× | bnb | training-friendly, not ideal for serving |
| **GGUF (Q4_K_M / Q5_K_M / ...)** | 4–8 | no | similar to AWQ at same bits | supported in vLLM | ~4× at Q4 | llama.cpp-compatible | useful for cross-stack portability |

> ⚠ AutoAWQ is **deprecated** as of 2025. For new AWQ checkpoints, use **llm-compressor**. [Source: [[sources/vllm-quantization-docs]]]

Headline rule for A10G: **AWQ INT4 (or GPTQ INT4) is the default for any model that doesn't fit comfortably at FP16/BF16.** INT8 is the conservative choice when an extra ~1pt of quality matters and the model still fits.

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

Open question. **All public BFCL / SWE-bench / LiveCodeBench / EvalPlus scores are FP16/BF16** ([Source: [[sources/bfcl-leaderboard-2026-05]]], [[sources/code-benchmarks-2026-05]]) — the only quantized public numbers in this domain are Aider's Ollama Q8 entries (Qwen2.5-Coder-32B Q8: 72.9 pass@2; Codestral 22B Q8: 48.1 pass@2 — comparable to API FP16).

- General benchmarks (MMLU, HumanEval) typically lose 0.5–1.5 points at INT4. *(AWQ paper claim — see [original AWQ paper](https://arxiv.org/abs/2306.00978))*
- **Tool-calling JSON compliance** is a structured-output task; constrained decoding ([[infrastructure/vllm]] §Structured outputs) compensates well at any quant.
- **Long-context retrieval** (needle-in-haystack) sometimes degrades at aggressive quant — cited concern, not always confirmed.
- **Multi-step agentic tasks** (e.g. SWE-bench-style) — unclear how much INT4 hurts. Worth testing per-model.

For seeded model pages, default recommendation is AWQ INT4 unless the model fits at FP16 on a single A10G.

## Recommended explicit flags

```
--quantization awq_marlin    # for AWQ-INT4 checkpoints
--quantization gptq_marlin   # for GPTQ-INT4 checkpoints
--kv-cache-dtype auto        # or fp8 to halve KV-cache memory at small quality cost
```

## Related
- [[hardware/a10g-g5xlarge]]
- [[infrastructure/vllm]]
- [[concepts/tool-selection]]
- [[concepts/code-generation]]

## Sources
- [[sources/vllm-quantization-docs]]
- [[sources/nvidia-a10g-specs]]
