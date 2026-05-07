---
tags: [hardware, gpu, aws]
last_updated: 2026-05-07
source_count: 3
---

# NVIDIA A10G on AWS g5.xlarge

The primary deployment target. A single A10G with 24 GB VRAM dictates almost every modeling choice in this wiki.

## A10G specs

| Property | Value | Source |
|---|---|---|
| Architecture | Ampere (sm_86) | [Source: [[sources/nvidia-a10g-specs]]] |
| VRAM | 24 GB GDDR6 | [Source: [[sources/nvidia-a10g-specs]]] |
| Memory bandwidth | 600 GB/s | [Source: [[sources/nvidia-a10g-specs]]] |
| FP16/BF16 tensor TFLOPS (dense) | **70** (vs ~125 for the data-center A10) | [Source: [[sources/nvidia-a10g-specs]]] |
| INT8 tensor TOPS | likely ~140 (proportional to A10's 250) | [Source: [[sources/nvidia-a10g-specs]]] |
| FP8 tensor cores | none (Ampere predates FP8) | [Source: [[sources/nvidia-a10g-specs]]] |
| RT cores | 80 | AWS G5 page |
| Tensor cores | 320 (3rd gen) | AWS G5 page |
| NVLink on g5 | none — PCIe Gen4 only | [Source: [[sources/nvidia-a10g-specs]]] |

> ⚠ Contradiction: AWS marketing for G5 quotes "up to 250 TOPS" — that aligns with the data-center A10, not the A10G. Hands-on testing finds A10G FP16 tensor compute is ~56% of A10's. Treat compute numbers as approximate.

Implications:
- **No native FP8.** Ampere predates FP8 tensor cores. vLLM's FP8 W8A8 path explicitly excludes Ampere — FP8 checkpoints load in a W8A16 fallback (memory savings, no compute speedup). Use BF16/FP16 baseline or quantize to INT4/INT8. See [[infrastructure/quantization]].
- **Memory-bandwidth-bound at decode.** A10G's 600 GB/s is the single biggest throughput limiter; **AWQ/GPTQ INT4 weight-only quant** (cutting decode bytes ~4×) is the largest single throughput win. Marlin kernels work well on Ampere. See [[infrastructure/quantization]].
- **No NVLink on g5.** Multi-GPU tensor parallelism on g5.12xlarge / g5.24xlarge / g5.48xlarge is over PCIe Gen4 only — meaningfully slower than A100/H100 NVLink boxes.

## g5.xlarge instance

| Property | Value | Source |
|---|---|---|
| GPUs | 1× A10G (24 GB) | [Source: [[sources/aws-ec2-pricing-2026-05]]] |
| vCPUs | 4 | AWS G5 page |
| RAM | 16 GiB | AWS G5 page |
| NVMe SSD | 1× 250 GB | AWS G5 page |
| Network | Up to 10 Gbps | AWS G5 page |
| **On-demand $/hr (us-east-1)** | **$1.006** | [Source: [[sources/aws-ec2-pricing-2026-05]]] |
| 24×7 monthly | ~$734 | derived (730 hr × $1.006) |
| Spot (typical) | 30–70% off on-demand | AWS Spot Advisor |

## g5 family pricing (us-east-1, on-demand) — single-A10G tier

| Instance | $/hr | Notes |
|---|---|---|
| g5.xlarge | $1.006 | cheapest single-A10G |
| g5.2xlarge | $1.212 | more vCPU/RAM, **same GPU** |
| g5.4xlarge | $1.624 | |
| g5.8xlarge | $2.448 | |
| g5.16xlarge | $4.096 | |

**Stepping up within the 1× A10G tier (g5.2xlarge → g5.16xlarge) buys more vCPU/RAM/NVMe/network — not more VRAM.** [Source: [[sources/aws-ec2-pricing-2026-05]]]

For multi-GPU options (g5.12xlarge and beyond), see [[hardware/multi-gpu-options]].

## VRAM budget on a single A10G

```
24 GB ≈ weights + KV cache + activations + framework overhead
                              ↑
                        the variable you tune
```

- **Framework overhead** (CUDA context, vLLM workers, kernels): ~1.5–2 GB.
- **Activations** (per request, scales with batch and sequence): ~0.5–2 GB at typical batch.
- **Weights** (the dominant fixed cost): see model pages and [[infrastructure/quantization]].
- **KV cache** = the rest. This is what limits concurrent requests and effective context.

vLLM defaults `--gpu-memory-utilization` to **0.92** as of 0.18+ (was 0.9 prior). [Source: [[sources/vllm-quantization-docs]]]

Approximate weight footprints:

| Params | FP16/BF16 | INT8 | AWQ INT4 / GPTQ INT4 |
|---|---|---|---|
| 7B | ~14 GB | ~7 GB | ~4 GB |
| 8B | ~16 GB | ~8 GB | ~5 GB |
| 13–14B | ~28 GB ❌ | ~14 GB | ~8–9 GB |
| 22–24B | ~44–48 GB ❌ | ~22–24 GB ⚠ | ~12–14 GB |
| 32B | ~64 GB ❌ | ~32 GB ❌ | ~18–19 GB ⚠ |
| 70B | ~140 GB ❌ | ~70 GB ❌ | ~40 GB ❌ |

✅ comfortable / ⚠ tight / ❌ won't fit on single A10G.

KV-cache size per token (FP16 KV): `2 × num_layers × num_kv_heads × head_dim × 2 bytes`.

For Llama-3-8B (32 layers, 8 KV heads, 128 dim): ~131 KB/token → **~131 MB per 1k tokens**. An 8k context costs ~1 GB of KV; 32k costs ~4 GB; 128k costs ~16 GB. GQA models (most modern ones) are far cheaper than MHA models per token.

## Practical max-context heuristic on 24 GB

For a single concurrent request (batch=1):

| Model size at quant | Weights | KV/1k tok | KV budget left | Practical max ctx |
|---|---|---|---|---|
| 7B AWQ INT4 | 4 GB | ~130 MB | ~18 GB | 100k+ tokens |
| 14B AWQ INT4 | 8 GB | ~250 MB | ~14 GB | ~50k tokens |
| 32B AWQ INT4 | 19 GB | ~400 MB | ~3 GB | ~7k tokens ⚠ |

These are batch-1 ceilings. With concurrent requests, divide. vLLM's `--max-model-len` and `--gpu-memory-utilization` flags are the primary knobs. [Source: [[sources/vllm-quantization-docs]]]

## Related
- [[hardware/multi-gpu-options]] — when to step up
- [[infrastructure/quantization]] — what fits at what quant
- [[infrastructure/vllm]] — KV-cache flags and tuning
- [[infrastructure/nvidia-dynamo]] — single-GPU relevance (low)

## Sources
- [[sources/aws-ec2-pricing-2026-05]]
- [[sources/nvidia-a10g-specs]]
- [[sources/vllm-quantization-docs]]
