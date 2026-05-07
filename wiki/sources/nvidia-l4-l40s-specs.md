---
tags: [source, hardware, gpu, nvidia]
source_path: raw/nvidia-l4-l40s-datasheet.md
source_url: https://www.nvidia.com/en-us/data-center/l40s/
ingested: 2026-05-07
last_updated: 2026-05-07
---

# NVIDIA L4 / L40S — Ada Lovelace inference GPUs (sm_89)

Both L4 and L40S are Ada Lovelace generation (sm_89) — the inference-optimized
generation between A10G (Ampere sm_86) and H100 (Hopper sm_90).

## Key claims

- **L4** (g6): 24 GB GDDR6, **300 GB/s** memory bandwidth, ~121 TF FP16 dense, FP8 native.
  Same VRAM as A10G but **half the memory bandwidth** — typically *slower* on
  decode-bound LLM inference despite the newer arch and FP8 support.
- **L40S** (g6e): **48 GB GDDR6**, **864 GB/s** memory bandwidth, ~362 TF FP16 dense,
  FP8 native. The biggest single-GPU VRAM available on AWS G-class — **g6e.xlarge at $1.861/hr
  is the cheapest 48 GB single-GPU box on AWS**.
- Both support vLLM `--quantization fp8` (model-side FP8 weights) — A10G cannot.
- Neither has NVLink: multi-GPU on g6/g6e is PCIe-Gen4 only.

## When L4 / L40S beats A10G for LLM inference

- **L40S** consistently beats A10G: more VRAM, more bandwidth, more compute, FP8.
  The only reason to stay on A10G is cost ($1.006 vs $1.861/hr at xlarge).
- **L4** is a sidegrade or downgrade vs A10G for LLM decode. Wins on $/hr at xlarge
  ($0.80 vs $1.006), wins for vision and prefill-heavy mixed workloads, loses on
  generation throughput at long context.

## Pages updated on ingest

- [[hardware/aws-gpu-landscape]] — added L4 and L40S rows
- [[hardware/g6e-l40s]] — new dedicated page
- [[infrastructure/quantization]] — note FP8 unlocked on Ada (L4/L40S)
