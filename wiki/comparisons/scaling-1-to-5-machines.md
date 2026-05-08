---
tags: [comparison, scaling, multi-node, multi-replica]
last_updated: 2026-05-08
source_count: 5
---

# Scaling from 1 → 2 → 5 machines

The user-facing question: "I have one g5.xlarge with vLLM serving a model.
What changes when I add a second machine, and how do I keep going to five?"
This page is the practical recipe — substrate-aware (g5 is EFA-less), with
specific commands, observability, and a measurement plan.

## TL;DR

| Step | Topology | Stack | Why this shape |
|---|---|---|---|
| **1 machine** | 1× g5.xlarge, single vLLM | [[infrastructure/vllm]] | Wiki baseline. No orchestration needed. |
| **2 machines** | 2× g5.xlarge, **independent replicas** behind a router | [[infrastructure/vllm]] + [[infrastructure/vllm-production-stack]] (or [[infrastructure/llm-d]] / [[infrastructure/aibrix]]) with **prefix-aware routing** | g5 has no EFA; TP across nodes is non-viable. Each replica is one g5; router does KV-cache-aware load balancing. |
| **5 machines** | 5× g5.xlarge, same shape | Same as 2-machine | Linear-ish scaling; watch for routing imbalance and cache-hit-rate variance across replicas. |

⚠ **Do not scale by stepping into multi-node TP on g5**. EFA is unavailable
on g5; ENA-only at 25 Gbps cannot service per-layer AllReduce traffic. If
your model crosses a single g5.xlarge's 24 GB VRAM (even at AWQ INT4),
move to **g6e.xlarge** (48 GB single-GPU, EFA-capable) before considering
multi-node.

## The substrate constraint

[Source: [[sources/aws-efa-multinode-2026-05]]]

| AWS instance | EFA? | NVLink? | Verdict for cross-node TP |
|---|---|---|---|
| g5.xlarge | **No** | No | TP across nodes infeasible |
| g5.48xlarge | **No** | No (NVSwitch absent) | TP within node only |
| g6e.xlarge | Yes | No | TP across nodes viable |
| p4d/p5/p5e/p5en | Yes (400–3,200 Gbps) | NVSwitch | Full multi-node TP |

This is why the canonical multi-machine answer for this wiki is **independent
replicas behind a KV-aware router**, not multi-node TP.

For a deeper discussion of when each parallelism axis applies, see
[[concepts/parallelism-strategies]].

## Stage 1 — single g5.xlarge baseline

Already covered by the wiki — see [[hardware/a10g-g5xlarge]],
[[infrastructure/vllm]]. Pick a model from
[[comparisons/tool-calling-models-on-a10g]]. Example for
[[models/qwen3-coder-30b-a3b]] (Apache-2.0, MoE 31B/3.3B):

```bash
vllm serve Qwen/Qwen3-Coder-30B-A3B-Instruct-AWQ \
  --quantization awq_marlin \
  --max-model-len 32768 \
  --enable-auto-tool-choice \
  --tool-call-parser qwen3_coder \
  --port 8000
```

Establish performance baseline with [[concepts/serving-performance-measurement]]:

```bash
# GenAI-Perf concurrency sweep
for c in 1 2 5 10 50 100 250; do
  genai-perf profile -m Qwen3-Coder-30B \
    --service-kind openai --endpoint-type chat \
    --streaming -u localhost:8000 \
    --concurrency $c \
    --measurement-interval 30000 \
    --synthetic-input-tokens-mean 550 --synthetic-input-tokens-stddev 150 \
    --output-tokens-mean 150 \
    --extra-inputs ignore_eos:true \
    --tokenizer Qwen/Qwen3-Coder-30B-A3B-Instruct
done
```

Record:
- TTFT p50/p95/p99 vs concurrency.
- ITL p50/p95/p99 vs concurrency.
- RPS at saturation.
- **Goodput** under your SLO (e.g., MLPerf Llama 2 Interactive: TTFT_p99 ≤
  450 ms, TPOT_p99 ≤ 40 ms — adjust as needed).

This is the per-replica capacity you'll multiply by 2 (then by 5) — minus
routing overhead and minus per-replica imbalance.

## Stage 2 — 2 machines

### Topology

```
                      ┌─────────────┐
client (OpenAI ───►   │  Router     │
   API client)        │ (vllm-router│
                      │  / llm-d    │
                      │  / AIBrix)  │
                      └─────┬───────┘
                            │ KV-aware / prefix-aware
                ┌───────────┴────────────┐
                │                        │
        ┌───────▼───────┐        ┌───────▼───────┐
        │ g5.xlarge #1  │        │ g5.xlarge #2  │
        │ vLLM replica  │        │ vLLM replica  │
        └───────────────┘        └───────────────┘
```

### Why a router (and not Round Robin)

Naive round-robin distributes requests evenly but **destroys prefix-cache
reuse**. For agent / tool-calling / RAG workloads (the wiki's focus), every
request shares a long system prompt + tool schema. Without prefix-aware
routing, the second replica processes the prefix from scratch — its TTFT
matches a cold-cache cluster. With prefix-aware routing, the request goes
to the replica that already has that prefix cached.

llm-d's measured impact: **TTFT P90 0.542 s vs 31.083 s** (precise
prefix-aware vs approximate). 57× win.
[Source: [[sources/k8s-llm-orchestration-2026-05]]]

### Four valid implementations

| Stack | When to choose | Setup |
|---|---|---|
| [[infrastructure/vllm-production-stack]] | Default; lightest path; Helm-deployable | `helm install vllm vllm/vllm-stack -f values.yaml` |
| [[infrastructure/llm-d]] | Already on KServe; need full operator (canary, scale-to-zero) | Apply `LLMInferenceService` CRD |
| [[infrastructure/aibrix]] | Heterogeneous GPU pool (mix g5 + g6e); high-density LoRA | K8s deploy via aibrix manifests |
| [[infrastructure/ray-serve-llm]] | Multi-LoRA + multi-model + Pythonic compound pipelines | `build_openai_app(...)` Python def |

For the **default 2 → 5 g5 case**, vLLM Production Stack is the right choice.
Helm chart skeleton:

```yaml
# values-2node.yaml
servingEngineSpec:
  modelSpec:
    - name: qwen3-coder-30b
      modelURL: Qwen/Qwen3-Coder-30B-A3B-Instruct-AWQ
      replicaCount: 2
      requestCPU: 4
      requestMemory: "16Gi"
      requestGPU: 1
      vllmConfig:
        maxModelLen: 32768
        toolCallParser: qwen3_coder
        enableAutoToolChoice: true
        quantization: awq_marlin

routerSpec:
  replicaCount: 2
  routingLogic: prefixaware
  enableObservability: true
```

```bash
helm install vllm vllm/vllm-stack -f values-2node.yaml
```

The router is itself replicated for HA; replicas are the vLLM pods (one per
g5.xlarge). Service discovery is via the K8s API.

### Validating the 2-node setup

Re-run the GenAI-Perf sweep against the **router endpoint**, not against an
individual replica:

```bash
genai-perf profile ... -u <router-svc>:8000 --concurrency $c ...
```

Track:
- **Cluster goodput** at the router endpoint vs sum-of-replicas.
- **Per-replica request count distribution** (Gini / max-min ratio).
- **Per-replica prefix-cache hit rate** (Prometheus
  `vllm:prefix_cache_hits_total` / `vllm:prefix_cache_queries_total`).
- **Inter-replica TTFT divergence** (p99 max vs p99 min).

Expect **1.6×–1.9× the single-replica RPS** at the same SLO (not 2.0×).
The shortfall is router overhead + per-replica imbalance + cold-cache cost
on the second replica during prefix-cache fill.

If you see < 1.5×, investigate:
- Routing algorithm — is it actually prefix-aware?
- Imbalance — is one replica getting 70% of traffic?
- Cache hit rate — is it consistent across replicas? (If hit rate is 90%
  on replica A and 30% on replica B, prefix routing is over-pinning to A.)

## Stage 3 — 5 machines

### Topology

Same shape as 2 machines, just more replicas. The router gets a 5-replica
backend pool.

```yaml
# values-5node.yaml
servingEngineSpec:
  modelSpec:
    - name: qwen3-coder-30b
      replicaCount: 5
      ...
```

### What you should measure

- **Goodput** at the router. Plot 1, 2, 3, 4, 5 replicas vs goodput at your
  SLO. Linear scaling means each added replica adds ~1× single-replica
  goodput.
- **Variance across replicas**. Measure per-replica request count, p99
  TTFT, cache hit rate. A well-routed cluster has <20% variance across
  replicas. >50% means the router is misbehaving or your workload has
  inherent skew.
- **Tail latency under load**. Sweep the cluster to its goodput ceiling.
  Past that, p99 TTFT spikes; that's your operating ceiling.

### Failure mode to watch

Beyond ~3 replicas, prefix-aware routing can over-pin specific replicas
to specific prefixes, causing imbalance. Mitigation:
- Bound the prefix-aware-routing affinity (Production Stack and llm-d both
  expose policies to fall back to round-robin when a replica is overloaded).
- Allow KV cache duplication across replicas — accept the cache-storage
  cost for better load balance.

## When to step out of "g5 replicas + router"

| Condition | Move to |
|---|---|
| Model > 24 GB even at AWQ INT4 | [[hardware/g6e-l40s\|g6e.xlarge]] (48 GB single-GPU) |
| Need cluster-wide concurrency > ~5×g5 can serve | g6e.xlarge replicas (48 GB), or move to TP within multi-GPU instance |
| Need disaggregated prefill/decode | g6e or p4d/p5 (need EFA) + [[infrastructure/nvidia-dynamo]] |
| Single very large model crossing single-instance VRAM | [[infrastructure/leaderworkerset]] on EFA-capable instance class |

The full size-up decision tree is in [[hardware/multi-gpu-options]] and
[[hardware/aws-gpu-landscape]].

## Comparison with multi-node TP/PP (when EFA is available)

For instances with EFA (g6e+ on AWS), you have a second option for scaling:
multi-node tensor / pipeline parallelism, fronted by a single API server.

[Source: [[sources/vllm-distributed-serving-2026-05]]]

```bash
# Head node (g6e.48xlarge, 8× L40S)
vllm serve <model> --tensor-parallel-size 8 --pipeline-parallel-size 2 \
  --nnodes 2 --node-rank 0 --master-addr <HEAD>

# Worker node (g6e.48xlarge, 8× L40S)
vllm serve <model> --tensor-parallel-size 8 --pipeline-parallel-size 2 \
  --nnodes 2 --node-rank 1 --master-addr <HEAD> --headless
```

This shape gives you **one logical replica** spanning two physical nodes,
serving a model that wouldn't fit on either alone. **Different problem from
"more QPS"**:

| Goal | Topology |
|---|---|
| **More QPS** for a model that fits one node | Independent replicas + router |
| **Larger model** that doesn't fit one node | Multi-node TP+PP, one replica |
| **Both** | Independent replicas, each multi-node |

Ranking by simplicity (easiest first): single-replica → multi-replica →
multi-node single-replica → multi-replica multi-node.

## Performance measurement plan (full)

For each topology (1, 2, 5 replicas), record:

1. **GenAI-Perf concurrency sweep** at 1, 2, 5, 10, 50, 100, 250 concurrency.
2. **Goodput** at your declared SLO (use MLPerf Llama 2 Interactive defaults
   if you have none: 99p TTFT ≤ 450 ms, 99p TPOT ≤ 40 ms).
3. **Distributions** — never just means. P50, P90, P95, P99 for TTFT and ITL.
4. **Prefix-cache hit rate** per replica (Prometheus PromQL).
5. **Per-replica request distribution** (Gini coefficient or max/min ratio).
6. **Queue time** (`vllm:request_queue_time_seconds`) — separates queueing
   from compute.

⚠ **Use open-loop load generation** for SLO sizing (`--request-rate R` in
GenAI-Perf). Closed-loop tests auto-throttle and mask saturation. See
[[concepts/serving-performance-measurement]] (Open-loop vs closed-loop).

## Cost calculation example

Suppose:
- g5.xlarge: $1.006/hr on-demand.
- Single-replica goodput at SLO: ~5 RPS (workload-dependent — measure!).
- Mean ISL/OSL: 550/150 tokens.

Per-machine yearly cost = `1.006 × 24 × 365 × 0.97` (97% uptime) ≈ $8,549.
Per-machine yearly tokens = `5 × (550+150) × 3600 × 24 × 365 × 0.97`
≈ 107 G tokens.
Per-machine $/Mtok ≈ $0.080 mixed.

Scaling to 5 replicas:
- 5 g5 cost: ~$42,743/yr.
- Goodput at 5 replicas (assuming 0.85 scaling efficiency): ~21 RPS.
- 5-replica $/Mtok ≈ $0.094 (slightly worse than 1× due to router + imbalance).

If your goodput scaling drops to 0.6× per replica, $/Mtok rises to $0.13 —
investigate routing imbalance.

## Related

- [[infrastructure/vllm]]
- [[infrastructure/vllm-production-stack]] (canonical implementation)
- [[infrastructure/llm-d]] (operator alternative)
- [[infrastructure/aibrix]] (heterogeneous-pool alternative)
- [[infrastructure/ray-serve-llm]] (Pythonic alternative)
- [[infrastructure/leaderworkerset]] (for multi-node single-replica)
- [[concepts/parallelism-strategies]]
- [[concepts/serving-performance-measurement]]
- [[concepts/disaggregated-serving]]
- [[hardware/aws-efa]]
- [[hardware/multi-gpu-options]]
- [[comparisons/g6e-xlarge-deployment-recipe]] (next-step single-GPU upgrade)

## Sources

- [[sources/vllm-distributed-serving-2026-05]]
- [[sources/aws-efa-multinode-2026-05]]
- [[sources/k8s-llm-orchestration-2026-05]]
- [[sources/llm-serving-perf-metrics-2026-05]]
- [[sources/llm-serving-perf-tools-2026-05]]
