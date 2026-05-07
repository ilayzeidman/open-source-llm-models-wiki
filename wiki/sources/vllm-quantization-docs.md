---
tags: [source, infrastructure, vllm, quantization]
source_path: raw/vllm-quantization-docs-2026-05.md, raw/vllm-quantization-ampere.md
source_url: https://docs.vllm.ai/en/latest/features/quantization/
ingested: 2026-05-07
last_updated: 2026-05-07
---

# vLLM quantization docs (2026-05, vLLM 0.20.x)

## Key claims

### Support matrix on Ampere (sm_80/86 — A10G is sm_86)

| Format | Bits | A10G status | Kernel | Notes |
|---|---|---|---|---|
| AWQ | W4A16 | ✅ recommended | `awq_marlin` | Best perf/memory tradeoff for 14B–30B on A10G |
| GPTQ | W4A16 | ✅ | `gptq_marlin` | Equivalent to AWQ on Ampere |
| INT8 W8A8 | 8 | ✅ | cutlass | Conservative ~2× compression |
| **FP8 W8A8** | 8 | **❌ compute fallback** | — | "FP8 computation requires compute capability ≥ 8.9" — verbatim from docs. Ampere is 8.0/8.6. |
| FP8 weight-only (W8A16) | 8 wt / 16 act | ✅ silent fallback | FP8 Marlin | Loads FP8 checkpoints in this mode — saves weight memory, no compute speedup |
| BitsAndBytes | 4/8 | ✅ | bnb | Slower than AWQ for serving |
| GGUF | various | ✅ | llama.cpp-compatible | |
| INT4 W4A16 (compressed-tensors) | 4 | ✅ | Marlin | |

### Bottom line for A10G
- **AWQ INT4 with `awq_marlin`** is the recommended quant for any model that doesn't fit at FP16/BF16.
- **GPTQ INT4** is equivalent in quality and speed.
- FP8 checkpoints load on A10G but don't get H100-class compute speedup — only the weight-memory savings.
- AutoAWQ has been deprecated; use **llm-compressor** for new AWQ checkpoints.

### Structured-decoding backends

- **Default**: `auto` (resolves to `xgrammar` for almost all tool-call schemas).
- **xgrammar** is recommended for tool-call format compliance — best sustained throughput.
- **guidance** is preferred when TTFT matters more than steady-state TPS.
- **outlines** still supported but de-prioritized.
- **lm-format-enforcer** uses a different (Python `re`) regex dialect — not recommended for new deployments.
- CLI: `--guided-decoding-backend {auto,xgrammar,guidance,outlines,lm-format-enforcer}` (legacy). Canonical (0.18+): `--structured-outputs-config.backend=<name>`.

### Memory / serving flags (confirmed defaults)

| Flag | Default | Purpose |
|---|---|---|
| `--gpu-memory-utilization` | **0.92** (was 0.9 prior to ~0.18) | Fraction of GPU mem for executor + KV cache |
| `--max-model-len` | auto | Total context (`1k`/`25.6k`/`auto`/`-1` accepted) |
| `--max-num-seqs` | scheduler-dependent | Max concurrent sequences per iter |
| `--swap-space` | 4 GiB | CPU swap for KV-cache offload |
| `--enforce-eager` | False | Disable CUDA graph |
| `--enable-prefix-caching` | on by default in V1 (since ~0.16) | Reuse KV across shared prefixes |
| `--enable-chunked-prefill` | on by default in V1 | Split prefill across iters |
| `--tensor-parallel-size` | 1 | TP group size |
| `--quantization` | None (autodetect) | Force a method; explicit `awq_marlin`/`gptq_marlin` on Ampere |
| `--kv-cache-dtype` | auto | `auto`/`fp8`/`fp8_e5m2`/`fp8_e4m3`/`int8` |

### Version notes

- **Latest stable: vLLM 0.20.1 (2026-05-03).**
- Pinning recommendation: `vllm==0.20.1` for production. Skip 0.20.0 mid-cycle.
- 0.20 breaking changes:
  - PyTorch 2.11, CUDA 13.0.2 default
  - **transformers >= 5 required**
  - Metric `vllm:prompt_tokens_recomputed` removed → `PrefillStats`
  - Pooler config rename: `logit_bias`/`logit_scale` → `logit_mean`/`logit_sigma`
  - **Petit NVFP4** and **Sparse24** quantization paths removed
  - `experts_int8` consolidated into FP8 online quantization frontend
  - `LLM.reward` deprecated → `LLM.encode`

## Pages updated on ingest
- [[infrastructure/vllm]]
- [[infrastructure/quantization]]
- [[hardware/a10g-g5xlarge]]
