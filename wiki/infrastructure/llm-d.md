---
tags: [infrastructure, serving, kubernetes, kserve, multi-node]
last_updated: 2026-05-08
source_count: 1
---

# llm-d

A Kubernetes Operator for production LLM serving on K8s, built jointly with
the [KServe](https://kserve.github.io/website/) community and Red Hat.
Apache-2.0. The "full-fat" alternative to the lighter
[[infrastructure/vllm-production-stack]] when you need an operator with CRD-
driven lifecycle, multi-model deployments, canary, and integration with
KServe's Inference Gateway. [Source: [[sources/k8s-llm-orchestration-2026-05]]]

## Releases

- 0.4 (2025)
- 0.5 (2026)
- **0.7** (Apr 2026, current)

## CRDs

| CRD | Purpose |
|---|---|
| `LLMInferenceService` | Per-model deployment surface (replicas, GPUs, runtime, route) |
| `LLMInferenceConfig` | Reusable configuration template |

## Architecture

llm-d integrates with the **KServe Inference Gateway Extension** (Envoy +
Gateway API Inference Extension) for KV-aware routing.

The **Endpoint Picker (EPP)** + `InferencePool` CRD scores model server pods
by: "real-time metrics, KV-cache affinity, and configured policies."

## Reported gains

- "**3× improvement in output tokens/s**"
- "**2× reduction in time to first token**" (vs round-robin baseline)

## KV-cache experiment (16-H100 cluster, B2B workload)

[Source: llm-d "KV-Cache Wins You Can See" blog]

Workload: 150 customers × 6,000-token contexts × 5 concurrent users.

| Routing | TTFT P90 |
|---|---|
| **Precise prefix-aware** | **0.542 s** |
| **Approximate** routing | **31.083 s** |

> "**precise-scheduling is 57× faster than approximate-scheduling.**"

Token cost ratio: "The cost for processing tokens already in the cache is
**10× lower** than for uncached tokens ($0.30 vs $3.00 per million)."

Hit-rate measurement is therefore a direct cost lever.

## Effective Cache Throughput

The blog introduces this metric: "the number of prompt tokens per second
served directly from the cache. This metric quantifies the computational work
the GPUs avoided." This is the right metric for evaluating any KV-aware
router. See [[concepts/serving-performance-measurement]].

## When to use llm-d

- You're standardizing on KServe (predictive + generative under one platform).
- You need full operator semantics (CRD watches, finalizers, canary, scale-
  to-zero, multi-tenancy).
- You want unified observability with the rest of your KServe stack.
- You need precise prefix-aware routing tightly integrated with the K8s
  control plane.

## When NOT to use

- Lighter K8s deploy with just Helm + vLLM cluster →
  [[infrastructure/vllm-production-stack]].
- Heterogeneous GPU pools, high-density LoRA → [[infrastructure/aibrix]].
- Single very large model spread TP+PP across nodes →
  [[infrastructure/leaderworkerset]] (used by both llm-d and Production Stack
  internally for the multi-host case).

## llm-d vs vLLM Production Stack

Both ship vLLM cluster + KV-aware routing on K8s. Differences:

| | vLLM Production Stack | llm-d |
|---|---|---|
| Distribution | Helm chart | K8s Operator |
| CRDs | Few/none | `LLMInferenceService`, `LLMInferenceConfig` |
| Lineage | vLLM team / LMCache | Red Hat / KServe community |
| Inference Gateway | Production Stack's own router | KServe Inference Gateway (Envoy + Gateway API) |
| Predictive ML alongside | Out of scope | Native (KServe predictive runtimes) |

If your team already runs KServe for predictive ML (sklearn, XGBoost, ONNX),
llm-d is the natural extension. If you're starting greenfield with just
vLLM, Production Stack has lower setup overhead.

## A10G fit

✅ — llm-d on EKS with g5.xlarge replicas is a viable shape. The benefits
(prefix-aware routing, KV-cache scoring) are the same as Production Stack;
the differentiator is operator semantics, not engine-level performance.

## Related

- [[infrastructure/vllm-production-stack]] (lighter alternative)
- [[infrastructure/vllm]]
- [[infrastructure/serving-stack-landscape]]
- [[concepts/serving-performance-measurement]]
- [[comparisons/scaling-1-to-5-machines]]

## Sources

- [[sources/k8s-llm-orchestration-2026-05]]
