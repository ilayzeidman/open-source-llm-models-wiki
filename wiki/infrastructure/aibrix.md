---
tags: [infrastructure, serving, kubernetes, multi-node]
last_updated: 2026-05-08
source_count: 1
---

# AIBrix

Cloud-native control plane for vLLM clusters. Apache-2.0. Distinguishing
features: **heterogeneous GPU scheduling**, **high-density LoRA management**,
**distributed KV cache**. Less mature than [[infrastructure/vllm-production-stack]]
or [[infrastructure/llm-d]] but addresses use cases neither covers as well.
[Source: [[sources/k8s-llm-orchestration-2026-05]]]

## Tagline

> "An open-source initiative designed to provide essential building blocks
> to construct scalable GenAI inference infrastructure."

## Components

| Component | Purpose |
|---|---|
| **LLM Gateway and Routing** | Multi-model, multi-replica routing with model-aware policies |
| **High-Density LoRA Management** | Many LoRA adapters per GPU; hot-swap |
| **LLM App-Tailored Autoscaler** | Scales on real-time demand (token rate, queue depth) |
| **Distributed KV Cache** | "high-capacity, cross-engine KV reuse" — KV cache shared across replicas |
| **Heterogeneous Serving** | "cost-effective SLO-driven LLM inference using heterogeneous GPUs" |
| **Unified AI Runtime** | Sidecar handling metrics, model download, standardization |
| **GPU Hardware Failure Detection** | Proactive eviction of degraded nodes |

## Stack

Go + Python + TypeScript; deploys via Kubernetes.

## When to use AIBrix

- **Heterogeneous GPU pool** — mixing g5.xlarge / g6e.xlarge / p5 instances
  in the same cluster, with SLO-driven routing to the cheapest GPU that
  meets the latency target. AIBrix is the only one of the K8s control planes
  that explicitly supports this as a first-class concept.
- **High-density LoRA** — hundreds or thousands of adapters per shared base
  model.
- **Distributed KV cache across replicas** — the cache fabric works across
  engines, not just within one vLLM instance.

## When NOT to use

- Homogeneous GPU pool with a single vLLM model →
  [[infrastructure/vllm-production-stack]] is simpler.
- Need full KServe operator semantics → [[infrastructure/llm-d]].
- Single replica → just vLLM.

## A10G fit

✅ — works on g5.xlarge replicas. Less battle-tested than Production Stack or
llm-d on the wiki's primary substrate. The heterogeneous-pool scenario is
where AIBrix distinguishes itself; for pure A10G fleets, the value over
Production Stack is marginal.

## Related

- [[infrastructure/vllm-production-stack]]
- [[infrastructure/llm-d]]
- [[infrastructure/ray-serve-llm]]
- [[infrastructure/serving-stack-landscape]]
- [[comparisons/scaling-1-to-5-machines]]

## Sources

- [[sources/k8s-llm-orchestration-2026-05]]
