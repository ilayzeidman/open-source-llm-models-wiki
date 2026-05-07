---
tags: [infrastructure, serving, nvidia-dynamo]
last_updated: 2026-05-07
source_count: 1
---

# NVIDIA Dynamo

NVIDIA's open-source orchestration layer that sits *above* engines like [[infrastructure/vllm]] (and TensorRT-LLM, SGLang, llama.cpp). Built in Rust + Python; targets datacenter-scale, multi-node distributed inference. OpenAI-compatible HTTP frontend.

## What Dynamo adds on top of vLLM

[Source: [[sources/nvidia-dynamo-readme]]]

| Feature | What it does | Single-A10G value |
|---|---|---|
| **Disaggregated prefill/decode** | Independent GPU pools per phase, each with its own parallelism strategy. Prefill is compute-bound; decode is memory-bandwidth-bound — splitting them improves utilization. | Limited — no second GPU to put prefill on. Some benefit possible by splitting prefill/decode into separate processes on the same GPU, but not the headline win. |
| **KV-cache-aware smart router** | Radix-tree-based; routes a request to the worker with the best cache-overlap score. | Moot at single-replica. Becomes valuable when running ≥2 replicas behind a load balancer. |
| **NIXL** (NVIDIA Inference Xfer Library) | Point-to-point comms across GPU/CPU/SSD/network tiers; supports NVLink, IB, RoCE, Ethernet. | Single-GPU: irrelevant. |
| **GPU Planner** | SLA-driven autoscaler that right-sizes prefill vs decode pools. | Multi-pool: irrelevant single-GPU. |
| **KV Block Manager (KVBM)** | Tiered KV-cache offload (GPU → CPU → SSD → remote). | g5.xlarge has 16 GiB RAM and a 250 GB NVMe — limited offload headroom but possible for long-context single-user workloads. |
| **ModelExpress** | GPU-to-GPU weight streaming for fast cold-start. | Single-GPU: irrelevant. |
| **Grove** | Kubernetes operator for topology-aware gang scheduling (NVL72-optimized). | Single-node: irrelevant. |
| **Engine-agnostic frontend** | OpenAI-compatible API in front of vLLM/TRT-LLM/SGLang/llama.cpp. | Useful as a stable API surface even on a single node. |

## Reported benchmarks (multi-GPU/multi-node)

- **30× throughput** on DeepSeek-R1 671B on GB200 NVL72.
- **>2×** on Llama-70B on Hopper.
- **2× faster TTFT** on Qwen3-Coder 480B.

These gains are categorically multi-GPU / multi-node. None translate to single-A10G.

## Honest assessment for g5.xlarge

> Dynamo's own README is direct about this: **"If you're running a single model on a single GPU, your inference engine alone is probably sufficient."** [Source: [[sources/nvidia-dynamo-readme]]]

On a single A10G, the Dynamo value-add is essentially:
- Stable OpenAI-compatible API front-end.
- Forward-compatibility — if you scale out to g5.12xlarge / g5.48xlarge / multi-node later, you don't change the client side.
- Optional KV offload to host RAM/NVMe for very long single-user contexts.

If single-node is the permanent target, **plain vLLM with the OpenAI server is simpler and gives ~the same single-GPU performance**. Dynamo earns its keep when you scale beyond one GPU.

## Architecture sketch

```
        ┌──────────────────────┐
client → │ Dynamo frontend (HTTP, OpenAI-compatible) │
        └─────────────┬────────┘
                      │ smart router (KV-cache aware)
            ┌─────────┴─────────┐
            │                   │
     ┌──────▼──────┐    ┌──────▼──────┐
     │ prefill     │    │ decode      │   ← can be same GPU or split
     │ worker(s)   │    │ worker(s)   │
     │ (vLLM)      │    │ (vLLM)      │
     └─────────────┘    └─────────────┘
            │                   │
            └────── NIXL ───────┘   ← KV-cache transfer
```

## Where to read further

- The hosted docs at `docs.nvidia.com/dynamo/...` returned 404s during research; the GitHub README is the most reliable architecture reference.

## Related
- [[infrastructure/vllm]]
- [[hardware/a10g-g5xlarge]]
- [[hardware/multi-gpu-options]]

## Sources
- [[sources/nvidia-dynamo-readme]]
