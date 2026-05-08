---
tags: [infrastructure, serving, vllm, kubernetes, multi-node]
last_updated: 2026-05-08
source_count: 1
---

# vLLM Production Stack

The vLLM team's reference implementation for clustering vLLM on Kubernetes.
Apache-2.0; released January 2025. The lightest path from "single-node vLLM"
to a multi-replica vLLM cluster with KV-aware routing, LMCache offload, and
Prometheus + Grafana observability. [Source: [[sources/k8s-llm-orchestration-2026-05]]]

## What it is

> "an open-source reference implementation of an inference stack built on top
> of vLLM, designed to run seamlessly on a cluster of GPU nodes."

Three components:

1. **Serving Engine** — vLLM pods (one per replica).
2. **Request Router** — directs traffic by routing keys / session IDs / prefix
   to maximize KV cache reuse across replicas.
3. **Observability Stack** — Prometheus + Grafana with TTFT distribution, KV
   cache hit rate, running/pending requests per instance, per-instance QPS.

OpenAI-compatible at the cluster boundary — drop-in for raw vLLM clients.

## Routing algorithms

| Algorithm | Use |
|---|---|
| Round robin | Default load balancing |
| Session-ID stickiness | Per-session affinity |
| **Prefix-aware load balancing** | "ensures that subsequent requests with the same prompt prefix are routed to the same instance, maximizing KV cache utilization" |
| KV-cache-aware | "smarter instance picks based on cached tokenized prompt" |

For agent / tool-calling workloads with shared system prompts and tool
schemas across all requests, **prefix-aware routing** is the headline win.
See [[concepts/serving-performance-measurement]] for KV-cache hit rate metrics.

## Deployment

```bash
helm install vllm vllm/vllm-stack -f values-deepseek.yaml
```

Helm chart parameters cover model selection, replica count, GPU resources per
replica, router algorithm, LMCache enable, monitoring stack toggle.

## Performance claim

> "10× better performance with 3-10× lower response delay & 2-5× higher
> throughput" via prefix-aware routing + KV cache sharing across instances.

This is vendor-supplied; verify with `benchmarks/multi-round-qa/` (the bundled
harness simulating N concurrent users × M rounds with shared system prompts).

## Router v0.5 (Dec 2025) additions

- Hierarchical KV offloading
- Cache-aware LoRA routing
- Active-active HA, scale-to-zero autoscaling
- UCCL-based transport resilience
- Validated: ~3.1k tok/s/B200 decode (wide-EP), 50k output tok/s on 16×16
  B200 P/D topology, "order-of-magnitude TTFT reduction vs round-robin
  baseline."

vs llm-d (router release blog claim):
> "vLLM Router throughput is 25% higher than llm-d and 100% higher than K8s-
> native load balancer, and vLLM Router's TTFT is 2000 ms faster."

⚠ This is one team's measurement methodology. See
[[concepts/serving-performance-measurement]] on how to verify cross-router
claims.

## Benchmarking methodology

`benchmarks/multi-round-qa/` simulates **N concurrent users × M rounds per
user** with shared system prompts. Reports: QPS, prompt throughput,
generation throughput, TTFT.

**Limitation**: doesn't directly measure KV-cache hit rate, prefix sharing,
or per-replica imbalance — these come from Prometheus metrics
(`vllm:prefix_cache_hits`, `vllm:kv_cache_usage_perc`) scraped during the
run. Always pair the harness with Prometheus scraping for cluster-level
observability.

## When to use vLLM Production Stack

- You already run vLLM and want to scale 1 → 2 → 5+ replicas with prefix-aware
  routing.
- Your workload has a long shared system prompt or tool schema (agentic /
  RAG / multi-turn) — prefix routing pays for itself.
- You want a Helm chart, not a full K8s operator.
- You don't need predictive ML in the same control plane.

## When NOT to use

- You need a full K8s operator with multi-model lifecycle, canary, scale-to-
  zero, predictive + generative under one CRD → use
  [[infrastructure/llm-d]] (with KServe).
- You need multi-host TP+PP for a single very large model →
  [[infrastructure/leaderworkerset]] under raw vLLM/SGLang.
- You need heterogeneous GPU pools → [[infrastructure/aibrix]] or
  [[infrastructure/ray-serve-llm]].
- Single replica only → just vLLM, no orchestrator.

## A10G fit

✅✅ — Production Stack is the **canonical answer** to the wiki's
"2 → 5 g5 machines" question. Each replica is one g5.xlarge running vLLM;
the router does prefix-aware routing across replicas. EFA is not required
(traffic is HTTP / OpenAI-compatible between client → router and router →
replica). See [[comparisons/scaling-1-to-5-machines]] for the recipe.

## Related

- [[infrastructure/vllm]]
- [[infrastructure/llm-d]]
- [[infrastructure/aibrix]]
- [[infrastructure/ray-serve-llm]]
- [[infrastructure/serving-stack-landscape]]
- [[concepts/serving-performance-measurement]]
- [[comparisons/scaling-1-to-5-machines]]

## Sources

- [[sources/k8s-llm-orchestration-2026-05]]
