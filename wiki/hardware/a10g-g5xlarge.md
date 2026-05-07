---
tags: [hardware, gpu, aws]
last_updated: 2026-05-07
source_count: 0
---

# NVIDIA A10G on AWS g5.xlarge

The primary deployment target. A single A10G with 24 GB VRAM dictates almost every modeling choice in this wiki.

## A10G specs *(unverified — needs source)*

| Property | Value |
|---|---|
| Architecture | Ampere (sm_86) |
| VRAM | 24 GB GDDR6 |
| Memory bandwidth | ~600 GB/s |
| FP16 / BF16 tensor TFLOPS | ~125 |
| INT8 tensor TOPS | ~250 |
| FP8 tensor cores | ❌ none (Ampere predates FP8) |
| NVLink | ❌ none on g5 (PCIe only between GPUs on multi-GPU g5) |

Implications:
- **No native FP8.** Use BF16/FP16 baseline or quantize to INT4/INT8. See [[infrastructure/quantization]].
- **Memory-bandwidth-bound at decode.** A10G's ~600 GB/s is the single biggest throughput limiter; INT4 weight-only quant cuts decode bytes ~4× and is the largest single throughput win.
- **No NVLink on g5** — multi-GPU tensor parallelism is over PCIe, which is fine for inference but slower than A100/H100 boxes.

## g5.xlarge instance *(unverified — needs source)*

| Property | Value |
|---|---|
| GPUs | 1× A10G (24 GB) |
| vCPUs | 4 |
| RAM | 16 GiB |
| NVMe SSD | 1× 250 GB |
| Network | Up to 10 Gbps |
| On-demand price (us-east-1) | ~$1.006 / hr |
| 24×7 monthly | ~$734 |
| Spot (typical) | ~$0.30–$0.60 / hr |
| 1-yr Reserved (no upfront) | ~40% off on-demand |

The g5 family scales:
- `g5.xlarge` / `g5.2xlarge` / `g5.4xlarge` / `g5.8xlarge` / `g5.16xlarge` — **all 1× A10G** (more vCPU/RAM/NVMe but same GPU). Stepping up here does **not** add VRAM.
- `g5.12xlarge` — 4× A10G (96 GB total).
- `g5.24xlarge` — 4× A10G (96 GB total).
- `g5.48xlarge` — 8× A10G (192 GB total).

See [[hardware/multi-gpu-options]] for when to step up.

## VRAM budget on a single A10G

Useful mental model:

```
24 GB ≈ weights + KV cache + activations + framework overhead
                              ↑
                        the variable you tune
```

- **Framework overhead** (CUDA context, vLLM workers, kernels): ~1.5–2 GB.
- **Activations** (per request, scales with batch and sequence): ~0.5–2 GB at typical batch.
- **Weights** (the dominant fixed cost): see model pages and [[infrastructure/quantization]].
- **KV cache** = the rest. This is what limits concurrent requests and effective context.

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

These are batch-1 ceilings. With concurrent requests (batch > 1), divide. vLLM's `--max-model-len` and `--gpu-memory-utilization` flags are the knobs.

## Related
- [[hardware/multi-gpu-options]] — when to step up
- [[infrastructure/quantization]] — what fits at what quant
- [[infrastructure/vllm]] — KV-cache flags and tuning
- [[infrastructure/nvidia-dynamo]] — single-GPU relevance

## Sources
- (none yet)

## TODO / verify
- AWS EC2 g5 pricing page (current us-east-1 on-demand and spot)
- NVIDIA A10G datasheet (confirm BW and TFLOPS numbers)
- vLLM docs on KV-cache memory accounting
