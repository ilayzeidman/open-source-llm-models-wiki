---
tags: [source, infrastructure, dynamo]
source_path: raw/nvidia-dynamo-readme.md
source_url: https://github.com/ai-dynamo/dynamo
ingested: 2026-05-07
last_updated: 2026-05-07
---

# NVIDIA Dynamo — README & overview

Dynamo is NVIDIA's open-source orchestration layer above engines like vLLM, TensorRT-LLM, SGLang, and llama.cpp. Built in Rust + Python; targets datacenter-scale, multi-node distributed inference. OpenAI-compatible HTTP frontend.

## Key claims

### Headline features
- **Disaggregated prefill/decode** — independent GPU pools per phase, each with its own parallelism strategy.
- **KV-cache-aware smart router** — Radix-tree based; routes a request to the worker with the best cache-overlap score.
- **NIXL** (NVIDIA Inference Xfer Library) — point-to-point comms across GPU/CPU/SSD/network tiers; supports NVLink, IB, RoCE, Ethernet.
- **GPU Planner** — SLA-driven autoscaler that right-sizes prefill vs decode pools.
- **KV Block Manager (KVBM)** — tiered KV-cache offload (GPU → CPU → SSD → remote).
- **ModelExpress** — GPU-to-GPU weight streaming for fast cold-start.
- **Grove** — Kubernetes operator for topology-aware gang scheduling (optimized for NVL72).

### Reported benchmarks
- 30× throughput on DeepSeek-R1 671B on GB200 NVL72.
- >2× on Llama-70B on Hopper.
- 2× faster TTFT on Qwen3-Coder 480B.

### Single-GPU value: essentially none

Dynamo's own README is direct about this: **"If you're running a single model on a single GPU, your inference engine alone is probably sufficient."**

- Disaggregated prefill/decode requires ≥2 GPUs split into separate pools.
- Smart routing only matters across replicas.
- KVBM tiering adds complexity beyond what vLLM already does in-engine.

For a single g5.xlarge, **run vLLM directly.** Dynamo earns its keep multi-GPU / multi-node.

### Citation URLs
- https://github.com/ai-dynamo/dynamo
- https://developer.nvidia.com/blog/introducing-nvidia-dynamo-a-low-latency-distributed-inference-framework-for-scaling-reasoning-ai-models/
- https://www.nvidia.com/en-us/ai/dynamo/

The hosted docs at `docs.nvidia.com/dynamo/...` returned 404s during research — the README is the most reliable architecture reference.

## Pages updated on ingest
- [[infrastructure/nvidia-dynamo]]
- [[wiki/overview]]
