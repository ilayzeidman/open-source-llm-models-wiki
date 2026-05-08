---
tags: [infrastructure, serving, nvidia-dynamo]
last_updated: 2026-05-08
source_count: 3
---

# NVIDIA Dynamo

NVIDIA's open-source orchestration layer that sits *above* engines like
[[infrastructure/vllm]], [[infrastructure/tensorrt-llm]], and
[[infrastructure/sglang]]. Built in Rust + Python; targets datacenter-scale,
multi-node distributed inference. OpenAI-compatible HTTP frontend.

> ⚠ **Naming**: NVIDIA renamed Triton Inference Server to **Dynamo-Triton**
> (March 2025). The "Dynamo" brand now spans **two products**: the
> LLM-focused **NVIDIA Dynamo** described here, and the rebranded
> **Dynamo-Triton** (multi-framework general inference). See
> [[infrastructure/triton-vs-dynamo]] for the disambiguation.

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

[Source: [[sources/nvidia-dynamo-multinode-2026-05]]]

| Result | Hardware | Source type |
|---|---|---|
| 30× throughput on DeepSeek-R1 671B | GB200 NVL72 | NVIDIA launch (GTC 2025) |
| Doubled Llama performance at same GPU count | Hopper | NVIDIA newsroom |
| 2× faster TTFT on Qwen3-Coder 480B | (multi-node) | NVIDIA |
| **12,587 tok/s/GPU** on Kimi K2.5 NVFP4 8k/1k | GB200 NVL72 + Dynamo+vLLM | SemiAnalysis InferenceMAX (real hardware) |
| 4,021 tok/s/GPU on same model | Single B200 node | SemiAnalysis (real hardware) |
| 6× MoE throughput in medium-latency regime | GB200 NVL72 | NVIDIA simulation |

The ~3× single-node-vs-disaggregated win on real GB200 NVL72 hardware is the
strongest data point for Dynamo at scale. These gains are categorically
multi-GPU / multi-node. **None translate to single-A10G.**

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

## Operational gotcha: `/v1/models` answers empty during startup

The `dynamo.frontend` HTTP service comes up in seconds — well before any
worker has finished pulling or loading weights. Until a `dynamo.vllm`
worker registers itself in etcd, the frontend answers
`GET /v1/models` with HTTP **200** and `{"object":"list","data":[]}`.

Naive readiness probes that test for "HTTP 200 on `/v1/models`" therefore
fire as ready while the engine is still loading (and may yet fail the
KV-cache check; see [[concepts/kv-cache-and-context-length]]). For an
operationally meaningful readiness probe, require the response's `data`
array to be non-empty:

```bash
# good — only true once a worker has registered the model in etcd
curl -fsS http://localhost:8000/v1/models | grep -q '"id"'
```

This matters because the worker's startup is the slow part: image pull
(~5–10 min from NGC), HuggingFace weight download (~10–30 min for 30 B+
models), then vLLM engine init + torch.compile (~1–3 min) + the
KV-cache check. The frontend's "readiness" tells you nothing about that
chain.

## Disaggregated prefill/decode (the canonical Dynamo win)

[Source: [[sources/nvidia-dynamo-multinode-2026-05]],
[[sources/disaggregated-serving-papers-2026-05]]]

The headline architecture: prefill (compute-bound) and decode (memory-
bandwidth-bound) on separate GPU pools, with KV cache transferred via NIXL.
Three-step flow: prefill compute → KV transfer → decode.

- Decode workers also handle short prefills (route decisions made there).
- Conditional disaggregation: requests go remote only when prefill length >
  threshold AND prefill queue not overloaded.
- Runtime elasticity: "Workers and prefill workers can be added and removed at
  runtime without any system-level synchronization or overheads."

Theory: see [[concepts/disaggregated-serving]] for the underlying papers
(DistServe, Splitwise, Mooncake) and the goodput improvements they report.

## Multi-node deployment alternatives

If your deployment is multi-replica (not multi-node single-model), Dynamo
may be heavier than you need. Lighter-weight alternatives:

- [[infrastructure/vllm-production-stack]] — Helm-deployable vLLM cluster
  with KV-aware router; the right answer for "scale 1 → 5 vLLM replicas
  on K8s."
- [[infrastructure/llm-d]] — KServe-integrated K8s Operator; full canary +
  scale-to-zero + multi-model.
- [[infrastructure/aibrix]] — heterogeneous GPU pools, high-density LoRA.
- [[infrastructure/leaderworkerset]] — K8s API for multi-host TP+PP single
  replica (e.g. Llama-3.1-405B across 2× p5.48xlarge).

For the "1 → 2 → 5 g5.xlarge" common scaling case (small model fits a
single g5), use [[comparisons/scaling-1-to-5-machines]] — Dynamo is overkill.

## Where to read further

- The hosted docs at `docs.nvidia.com/dynamo/...` are now reliable as of 2026.
- See [[infrastructure/triton-vs-dynamo]] to disambiguate Dynamo vs
  Dynamo-Triton vs the original Triton Inference Server.

## Related
- [[infrastructure/vllm]]
- [[infrastructure/sglang]]
- [[infrastructure/tensorrt-llm]]
- [[infrastructure/triton-vs-dynamo]]
- [[infrastructure/serving-stack-landscape]]
- [[infrastructure/vllm-production-stack]]
- [[infrastructure/llm-d]]
- [[hardware/a10g-g5xlarge]]
- [[hardware/multi-gpu-options]]
- [[hardware/aws-efa]]
- [[concepts/parallelism-strategies]]
- [[concepts/disaggregated-serving]]
- [[concepts/serving-performance-measurement]]
- [[concepts/kv-cache-and-context-length]]
- [[comparisons/scaling-1-to-5-machines]]

## Sources
- [[sources/nvidia-dynamo-readme]]
- [[sources/nvidia-dynamo-multinode-2026-05]]
- [[sources/disaggregated-serving-papers-2026-05]]
