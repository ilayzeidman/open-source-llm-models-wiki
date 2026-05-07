---
tags: [infrastructure, serving, nvidia-dynamo]
last_updated: 2026-05-07
source_count: 0
---

# NVIDIA Dynamo

NVIDIA's open-source, distributed inference framework that sits *above* engines like [[infrastructure/vllm]] (and TensorRT-LLM, SGLang, llama.cpp). It targets datacenter-scale serving with features that are mostly multi-GPU / multi-node.

## What Dynamo adds on top of vLLM *(unverified — needs source)*

| Feature | What it does | Single-A10G value |
|---|---|---|
| **Disaggregated prefill/decode** | Run prefill workers and decode workers as separate processes/GPUs; queue requests through a smart router. Prefill is compute-bound; decode is memory-bandwidth-bound — splitting them improves utilization. | Limited — no second GPU to put prefill on. Some benefit possible by splitting prefill/decode into separate processes on the same GPU, but not the headline win. |
| **KV-cache-aware smart router** | Routes a request to the worker that already has its prefix in KV cache (much better cache hit rate than naive round-robin). | Moot at single-replica. Becomes valuable when running ≥2 replicas behind a load balancer. |
| **NIXL** (NVIDIA Inference eXchange Library) | High-throughput KV-cache transfer between workers. | Single-GPU: irrelevant. |
| **Multi-node scheduling** | Fleet-level placement, autoscaling. | Single-node: irrelevant. |
| **Engine-agnostic frontend** | OpenAI-compatible API in front of vLLM/TRT-LLM/SGLang/llama.cpp. | Useful as a stable API surface even on a single node. |
| **GPU memory hierarchy / KV offload** | Tiered KV cache (HBM → host RAM → NVMe). | g5.xlarge has 16 GiB RAM and a 250 GB NVMe — limited offload headroom but still possible for long-context single-user workloads. |

## Honest assessment for g5.xlarge

The bulk of Dynamo's value is multi-GPU / multi-node. On a single A10G, you get:

- A stable OpenAI-compatible API front-end.
- Forward-compatibility — if you scale out to g5.12xlarge / g5.48xlarge / multi-node later, you don't change the client side.
- Optional KV offload to host RAM/NVMe for very long single-user contexts.
- The smart router is moot until you run ≥2 replicas.

If single-node is the permanent target, **plain vLLM with the OpenAI server is simpler and gives you ~the same single-GPU performance**. Dynamo earns its keep when you scale beyond one GPU.

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

## Related
- [[infrastructure/vllm]]
- [[hardware/a10g-g5xlarge]]
- [[hardware/multi-gpu-options]]

## Sources
- (none yet)

## TODO / verify
- NVIDIA Dynamo public docs and architecture overview
- Dynamo + vLLM integration guide
- Benchmarks: Dynamo + vLLM vs plain vLLM on (a) single GPU, (b) 2× A10G, (c) 8× A10G
- NIXL design doc
