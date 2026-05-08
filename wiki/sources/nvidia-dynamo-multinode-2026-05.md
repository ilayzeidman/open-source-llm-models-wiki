---
tags: [source, infrastructure, dynamo, multi-node]
source_path: raw/nvidia-dynamo-multinode-2026-05.md
source_url: https://github.com/ai-dynamo/dynamo
ingested: 2026-05-08
last_updated: 2026-05-08
---

# NVIDIA Dynamo (multi-node, May 2026 update)

Refresh of the original [[sources/nvidia-dynamo-readme]] with the multi-node-
specific architecture, performance numbers, and AWS deployment guidance from
NVIDIA's 2025–2026 disclosures.

## Key claims

### What it is

"A high-throughput, low-latency open-source inference serving framework for
deploying generative AI and reasoning models in large-scale distributed
environments." Orchestration **layer above** vLLM/SGLang/TensorRT-LLM, not a
replacement engine.

### Architecture components

- **Frontend** — OpenAI-compatible API gateway.
- **Workers** — engine instances (vLLM/SGLang/TRT-LLM).
- **Smart Router (KV-aware)** — Radix-tree of KV blocks; "2× faster TTFT on
  large models like Qwen3-Coder 480B."
- **SLO Planner** — autoscaler "dynamically decides whether to use
  disaggregated or aggregated serving, or to shift GPU resources between
  prefill and decode phases." "80% fewer SLA breaches at 5% lower TCO"
  (Alibaba APSARA 2025).
- **NIXL** — five backends: RDMA/InfiniBand, RoCE via UCX, TCP fallback,
  NVMe-oF, S3-compatible object storage. Transfers KV "directly from VRAM
  of prefill engine to VRAM of decode engine," non-blocking.
- **KVBM (KV Block Manager)** — three layers; offload tier GPU HBM → CPU DRAM
  → local SSD → remote storage. Vast Data: 35 GB/s to single H100; WEKA:
  270 GB/s across 8 GPUs via RDMA zero-copy.
- **Grove** — gang scheduler for NVL72.
- **DGDR** — DynamoGraphDeploymentRequest CRD for K8s.
- **ModelExpress** — GPU-to-GPU weight streaming over NIXL/NVLink; "7× faster
  cold-start for new replicas."

### Disaggregated prefill/decode (design doc)

- "The prefill and decode phases of LLM requests have different computation
  characteristics and memory footprints."
- Three-step flow: prefill compute → KV transfer → decode.
- Decode workers also handle short prefills.
- Conditional disaggregation: requests go remote only when prefill length >
  threshold AND prefill queue not overloaded.
- Runtime elasticity: "Workers and prefill workers can be added and removed
  at runtime without any system-level synchronization or overheads."

### Performance claims

| Result | Hardware | Source |
|---|---|---|
| 30× throughput on DeepSeek-R1 | GB200 NVL72 (Blackwell) | GTC 2025 |
| Doubled Llama perf at same GPU count | Hopper | NVIDIA newsroom |
| 6× medium-latency MoE throughput (sim) | GB200 NVL72 | NVIDIA blog (simulated) |
| 12,587 tok/s/GPU (Kimi K2.5 NVFP4 8k/1k) | GB200 NVL72 + Dynamo+vLLM | SemiAnalysis InferenceMAX (real) |
| 4,021 tok/s/GPU (same model, no Dynamo) | Single B200 node | SemiAnalysis (real) |

The ~3× per-GPU win between single-node and disaggregated is the strongest
real-world data point for Dynamo at scale.

### Single-GPU value (verbatim)

> "If you're running a single model on a single GPU, your inference engine
> alone is probably sufficient." — Dynamo README

### AWS EKS deployment notes

- Works with NVIDIA GPU instances "P6, P5, P4d, P4de, G5, and G6."
- Karpenter "burst new g6 instances in less than 60 seconds."
- "EFA is used to talk between the GPU nodes within a single Availability
  Zone."
- Model storage: Amazon EFS (NFS-style RWX shared by all pods).

### Dynamo SLA model (Planner)

- TTFT requirements applying to prefill.
- ITL requirements guiding decode deployment choices.
- "Dynamic rate matching... resources are allocated based on load across the
  prefill and decode phases."

### Citation URLs

- https://github.com/ai-dynamo/dynamo
- https://developer.nvidia.com/blog/introducing-nvidia-dynamo-a-low-latency-distributed-inference-framework-for-scaling-reasoning-ai-models/
- https://docs.nvidia.com/dynamo/v-0-7-1/design-docs/disaggregated-serving
- https://developer.nvidia.com/blog/how-to-reduce-kv-cache-bottlenecks-with-nvidia-dynamo/
- https://aws.amazon.com/blogs/machine-learning/accelerate-generative-ai-inference-with-nvidia-dynamo-and-amazon-eks/
- https://developer.nvidia.com/blog/how-nvidia-gb200-nvl72-and-nvidia-dynamo-boost-inference-performance-for-moe-models/

## Pages updated on ingest

- [[infrastructure/nvidia-dynamo]]
- [[infrastructure/triton-vs-dynamo]] (NEW)
- [[concepts/disaggregated-serving]] (NEW)
- [[comparisons/scaling-1-to-5-machines]] (NEW)
