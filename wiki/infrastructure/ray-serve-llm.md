---
tags: [infrastructure, serving, ray-serve, anyscale]
last_updated: 2026-05-08
source_count: 1
---

# Ray Serve LLM

Anyscale's production-grade LLM-serving API built on Ray Serve, replacing
the legacy `RayLLM` project (now archived). Apache-2.0. Most useful when you
need orchestration above [[infrastructure/vllm]]: multi-LoRA, multi-model,
autoscaling on heterogeneous clusters, or compound AI pipelines defined in
Python. [Source: [[sources/serving-stacks-alternatives-2026-05]]]

## What it is

Introduced in Ray ≥ 2.44.0. Anyscale frames the positioning explicitly:

> "vLLM is only responsible for single model replicas, but for production
> deployments you often need an orchestration layer to autoscale, handle
> different fine-tuned adapters, handle distributed model-parallelism, and
> author multi-model, compound AI pipelines. Ray Serve is built to address
> the gaps that vLLM has for scaling and productionization."

Ray Serve LLM is **complementary** to vLLM, not competitive. It runs vLLM
internally as the engine.

## Features

- "Automatic scaling and load balancing"
- "Unified multi-node multi-model deployment"
- OpenAI compatibility
- "Multi-LoRA support with shared base models"
- Pythonic deployment API: `build_openai_app`, `LLMConfig`, `LLMServer`

## Recent additions (Nov 2025)

- `build_dp_deployment` for **wide expert parallelism** — MoE experts spread
  across many GPUs with replicated attention; auto-rank-assignment +
  synchronized communication.
- `build_pd_openai_app` for **prefill/decode disaggregation** — `PDProxyServer`
  issues prefill with `max_tokens=1` to populate KV cache, then hands off to
  decode deployment.

## When to choose Ray Serve LLM

- **Multi-LoRA serving** — many fine-tuned adapters sharing a base model.
- **Multi-model deployments** — different models behind one Pythonic
  deployment definition.
- **Autoscaling on heterogeneous GPU clusters** — Ray's scheduler handles
  mixed-GPU-type pools natively.
- **Compound AI pipelines** — chains of models / retrieval + LLM /
  preprocessing pipelines defined as Ray Serve deployments.

## When NOT to choose

- Single-model, single-GPU prototype → just [[infrastructure/vllm]].
- K8s-native operator with CRDs → [[infrastructure/llm-d]] or
  [[infrastructure/vllm-production-stack]] is more idiomatic.
- Multi-node single-model TP+PP → [[infrastructure/leaderworkerset]] is
  simpler.
- Disaggregated prefill/decode at scale → [[infrastructure/nvidia-dynamo]]
  has more mature PD-disagg primitives.

## Migration from RayLLM

The legacy `ray-project/ray-llm` GitHub repo is **archived**. Anyscale
maintains an explicit migration guide at
docs.anyscale.com/llms/serving/migration_guide_to_ray_serve_llm/.

## A10G fit

✅ — Ray Serve LLM works on A10G. The marginal value over plain vLLM appears
once you're past 1 model on 1 GPU: multi-LoRA, multi-model, autoscaling
across multiple instances. For the wiki's "2 → 5 g5 machines" scaling story,
Ray Serve LLM is one of three valid choices alongside
[[infrastructure/vllm-production-stack]] and [[infrastructure/aibrix]].

## Related

- [[infrastructure/vllm]]
- [[infrastructure/serving-stack-landscape]]
- [[infrastructure/vllm-production-stack]]
- [[infrastructure/llm-d]]
- [[infrastructure/aibrix]]
- [[comparisons/scaling-1-to-5-machines]]

## Sources

- [[sources/serving-stacks-alternatives-2026-05]]
