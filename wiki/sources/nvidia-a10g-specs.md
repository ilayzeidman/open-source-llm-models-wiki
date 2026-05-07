---
tags: [source, hardware, gpu]
source_path: raw/nvidia-a10-datasheet.md, raw/nvidia-a10g-vs-a10.md
source_url: https://www.nvidia.com/en-us/data-center/products/a10-gpu/
ingested: 2026-05-07
last_updated: 2026-05-07
---

# NVIDIA A10 / A10G specs

The A10G is AWS's variant of NVIDIA's A10 datacenter GPU. Same memory subsystem, **different per-precision tensor compute** (A10G ships with reduced tensor TFLOPS).

## Key claims

| Property | A10 (datacenter) | A10G (AWS variant) | Source |
|---|---|---|---|
| Architecture | Ampere, sm_86 | Ampere, sm_86 | NVIDIA spec sheets |
| VRAM | 24 GB GDDR6 | 24 GB GDDR6 | NVIDIA / AWS G5 page |
| Memory bandwidth | 600 GB/s | 600 GB/s | NVIDIA A10 datasheet (shared mem) |
| FP16/BF16 tensor TFLOPS (dense) | **125** | **70** | Baseten A10 vs A10G |
| INT8 tensor TOPS | 250 | likely ~140 (proportional) | A10 datasheet; A10G measured 56% of A10 |
| RT cores | 72 | 80 | NVIDIA / AWS G5 page |
| Tensor cores | 288 (3rd gen) | 320 (3rd gen) | NVIDIA / AWS G5 page |
| FP8 tensor cores | none | none | architecture (Ada/Hopper-only) |
| NVLink | optional | **none on AWS G5** | Lenovo A10 product guide; AWS makes no NVLink claim |

> ⚠ Contradiction: AWS's G5 marketing page quotes "up to 250 TOPS" for A10G — a number that aligns with the **A10**, not the A10G. Baseten's hands-on testing finds A10G FP16 tensor compute is ~56% of A10's. The 250 TOPS figure is likely an aspirational quote of the A10 spec, not measured on A10G. **Treat all A10G compute numbers as approximate.**

## Implications

- **Memory-bandwidth-bound at decode.** 600 GB/s is the largest single throughput limiter; INT4 weight-only quantization (cutting decode bytes ~4×) is the largest single throughput win on this card. See [[infrastructure/quantization]].
- **No FP8.** Only Ada (4090) and Hopper (H100) and later have native FP8 tensor cores. vLLM's FP8 W8A8 path explicitly excludes Ampere — checkpoints load in W8A16 fallback (memory savings, no compute speedup). See [[infrastructure/vllm]].
- **No NVLink on G5.** Multi-GPU G5 instances (g5.12xlarge, .24xlarge, .48xlarge) communicate over PCIe Gen4 only. Tensor parallelism works but with much higher communication overhead than A100/H100 NVLink boxes.

## Note on the AWS A10G datasheet PDF

AWS hosts a PDF datasheet at `d1.awsstatic.com/.../NVIDIA_AWS_A10G_DataSheet_FINAL_02_17_2022.pdf` that should contain authoritative A10G-specific numbers. The PDF is not text-extractable via web tooling; for precise A10G values, download and OCR locally.

## Pages updated on ingest
- [[hardware/a10g-g5xlarge]]
- [[infrastructure/quantization]]
