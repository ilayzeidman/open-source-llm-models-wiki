# NVIDIA Dynamo — Repo README + Architecture Summary

source_url: https://github.com/ai-dynamo/dynamo
secondary_urls:
  - https://developer.nvidia.com/blog/introducing-nvidia-dynamo-a-low-latency-distributed-inference-framework-for-scaling-reasoning-ai-models/
  - https://www.nvidia.com/en-us/ai/dynamo/
fetched: 2026-05-07

## What it is

NVIDIA Dynamo is a "datacenter-scale distributed inference serving framework." It is an orchestration layer that sits ABOVE inference engines (vLLM, TensorRT-LLM, SGLang) — it does not replace them. Built in Rust (performance) + Python (extensibility).

> "Dynamo is the orchestration layer above inference engines — it doesn't replace SGLang, TensorRT-LLM, or vLLM, it turns them into a coordinated multi-node inference system."

## Headline features

1. **Disaggregated prefill/decode** — separates prefill (compute-bound) and decode (memory-bound) onto independently scalable GPU pools, each with its own parallelism strategy.
2. **KV-aware smart router** — Radix-tree-based router that scores cache overlap across the worker fleet and routes requests to maximize KV-cache reuse. Reported 2× faster TTFT on Qwen3-Coder 480B.
3. **NIXL (NVIDIA Inference Xfer Library)** — low-latency point-to-point comms library for moving KV-cache data across heterogeneous tiers (GPU, CPU, SSD, networked storage). Supports NVLink, InfiniBand, RoCE, Ethernet.
4. **GPU Planner / SLA-driven autoscaler** — profiles workload and right-sizes prefill vs decode pools to meet TTFT/ITL SLOs at minimum TCO.
5. **KV Block Manager (KVBM)** — tiered KV-cache offloading across GPU → CPU → SSD → remote storage.
6. **ModelExpress weight streaming** — GPU-to-GPU weight transfer via NIXL/NVLink, "7× faster model startup" for cold replicas.

## Supported backends

| Engine        | Status                                                |
|---------------|-------------------------------------------------------|
| vLLM          | Comprehensive support across all core capabilities    |
| TensorRT-LLM  | Full feature parity including KVBM                    |
| SGLang        | Full support: disaggregated, KV-aware routing, planner|

Frontend: OpenAI-compatible API.

## Architecture / topology

- **Frontend** — OpenAI-compatible HTTP endpoint.
- **Workers (engines)** — vLLM/TRT-LLM/SGLang instances doing the actual inference.
- **Router / Planner** — coordinates request routing and pool autoscaling.
- **etcd** — distributed state.
- **NATS** — KV-cache event bus.
- **Grove** — Kubernetes operator for topology-aware gang scheduling (optimized for NVL72 racks, NUMA placement).
- Outside K8s: etcd + NATS deployable independently (Slurm, etc.).

## Single-GPU value assessment

> "If you're running a single model on a single GPU, your inference engine alone is probably sufficient." — Dynamo README

Disaggregated serving is the big architectural win, and it requires ≥ 2 GPUs (typically many more, with separate pools). On a single A10G in g5.xlarge, **Dynamo provides essentially no benefit** beyond running vLLM directly. Smart routing only matters with a fleet of replicas; KVBM offload to CPU/SSD could in principle help long-context single-GPU workloads but is overkill vs vLLM's built-in offloading and CPU-paged-attention.

The natural Dynamo deployment is multi-node H100/H200 (or NVL72) clusters serving large reasoning models like DeepSeek-R1.

## Reported benchmarks

- DeepSeek-R1 671B on GB200 NVL72: up to 30× request-throughput gain
- Llama 70B on Hopper: more than 2× throughput
- Qwen3-Coder 480B: 2× faster TTFT (via KV-aware routing)

(All NVIDIA marketing claims — verify if used in cost analysis.)
