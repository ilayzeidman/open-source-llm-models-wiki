---
tags: [concepts, disaggregated, serving, multi-node]
last_updated: 2026-05-08
source_count: 2
---

# Disaggregated serving

The architecture in which **prefill** (compute-bound prompt processing) and
**decode** (memory-bandwidth-bound token generation) run on **separate GPU
pools**, with KV cache transferred between them. Productionized in NVIDIA
Dynamo, vLLM PD-disagg, SGLang PD-disagg, and llm-d. Backed by three
foundational papers (DistServe, Splitwise, Mooncake) that all argue
co-locating prefill and decode is wasteful.

## The core insight

[Source: [[sources/disaggregated-serving-papers-2026-05]]]

LLM inference has two phases with **opposite** hardware requirements:

| Phase | Bottleneck | Saturation behavior |
|---|---|---|
| **Prefill** | Compute (FLOPs) | "a small batch of prefills or even a single long enough prefill will easily saturate GPU computation" — DistServe |
| **Decode** | Memory bandwidth | "needs a much bigger batch size to hit the compute bound, and is more easily subject to the memory bandwidth limit" — DistServe |

Co-locating both on the same GPU forces compromises: the prefill pegs the
SMs while the decode is starving for memory bandwidth (and vice-versa). It
also forces both phases onto the same GPU class, when the cheaper option
might be H100 prefill + L40S decode.

## The three foundational papers

[Source: [[sources/disaggregated-serving-papers-2026-05]]]

### DistServe (Zhong et al., OSDI 2024)

- arXiv: https://arxiv.org/abs/2401.09670
- Headline: "**Serve 7.4× more requests or 12.6× tighter SLO**" while
  maintaining latency for >90% of requests.
- Introduces **goodput** as the headline metric (see
  [[concepts/serving-performance-measurement]]).
- Per-workload vs vLLM:
  - Chatbot: **2.0× – 3.41× higher goodput**.
  - Code completion: 3.2× higher goodput, 1.5× tighter SLO.
  - Summarization: **4.48× higher goodput**, 10.2× tighter SLO.

### Splitwise (Microsoft, ISCA 2024)

- arXiv: https://arxiv.org/abs/2311.18677
- Headline: "**1.4× higher throughput at 20% lower cost**; 2.35× throughput
  at same cost/power."
- Hardware insight: "Token generation phases do not require the compute
  capability of the latest GPUs, and can be run with lower power and cost."
- This is the academic basis for Dynamo's heterogeneous prefill/decode path.

### Mooncake (Moonshot AI, July 2024)

- arXiv: https://arxiv.org/abs/2407.00079
- Headline: "**Up to a 525% increase in throughput**" simulated; **75% more
  requests** in production at Kimi.
- Distinctive contribution: KV cache as a first-class **distributed**
  resource. Conceptual seed for Dynamo's KVBM and llm-d's KV-aware routing.
- Particularly effective for long-context with shared prefixes (RAG, agents).

## Productionization map

| Theory paper | Production system |
|---|---|
| DistServe | [[infrastructure/nvidia-dynamo]]; vLLM PD-disagg flag set; [[infrastructure/sglang]] PD-disagg |
| Splitwise (heterogeneous prefill/decode hardware) | Dynamo Planner; AIBrix heterogeneous serving |
| Mooncake (KV as distributed resource) | Dynamo KVBM + NIXL; llm-d's prefix-aware routing |

## What disaggregation buys you (numbers)

[Source: [[sources/nvidia-dynamo-multinode-2026-05]]]

### NVIDIA Dynamo on GB200 NVL72 (real hardware, SemiAnalysis InferenceMAX)
- GB200 NVL72 + Dynamo+vLLM: **12,587 tok/s/GPU** on Kimi K2.5 NVFP4 8k/1k.
- Single B200 node (no Dynamo): **4,021 tok/s/GPU**.
- **~3× per-GPU win** at scale from disaggregation.

### NVIDIA simulation (different methodology, less directly comparable)
- "6× throughput performance gain... in the medium latency regime" for
  DeepSeek-R1.
- "up to 3×" for Llama 70B.

## When does it help?

⚠ Per-paper numbers ranging "1.4×–525×" are for **specific workloads**
(model size, ISL/OSL ratio, SLO). Don't expect 7× on your workload.

Disaggregation pays off when:

1. **Long prefills** (RAG, agents with large tool schemas, long contexts)
   stall a co-located system.
2. **Tight TPOT SLO** at high concurrency (decode pool needs to be
   appropriately sized; prefill should not preempt).
3. **Large MoE models** where each phase has different optimal parallelism.
4. **Heterogeneous GPU pool** is available (cheap memory-bandwidth GPU for
   decode, peak-compute GPU for prefill).

It does **not** pay off when:

1. Prefills are short and uniform (chat with no system prompt).
2. Workload is single-replica on a single GPU (Dynamo's own README:
   "If you're running a single model on a single GPU, your inference engine
   alone is probably sufficient.").
3. Inter-node bandwidth is too low to transfer KV cache fast enough — the
   KV transfer becomes the new bottleneck.

## Implementation surfaces (May 2026)

| Engine / Stack | Disagg support |
|---|---|
| vLLM | First-class via PD-disagg flags + Dynamo integration |
| SGLang | First-class; LMSYS Jan 2026 chunked PP further extends |
| TensorRT-LLM | "Connector API for state transfer in disaggregated serving" (v1.1) |
| NVIDIA Dynamo | The reference orchestrator — where most users will encounter disagg |
| Ray Serve LLM | `build_pd_openai_app` (Nov 2025) |
| llm-d | KV-aware Inference Gateway routing complements disagg |
| vLLM Production Stack | Helm-deployable disagg topology in v0.5+ |

## A10G fit

⚠ **Disaggregated serving on a single g5.xlarge is irrelevant** — only one
GPU. On a multi-g5 fleet, disagg across g5 instances is hampered by lack of
EFA: KV transfer over ENA at ≤25 Gbps is the new bottleneck. Disaggregation
on AWS realistically requires g6e or higher (EFA-capable). See
[[hardware/aws-efa]].

For the wiki's primary "1 → 2 → 5 g5 machines" question, disaggregation is
**not** the answer. Independent replicas with prefix-aware routing is.

## Related

- [[infrastructure/nvidia-dynamo]]
- [[infrastructure/vllm]]
- [[infrastructure/sglang]]
- [[concepts/parallelism-strategies]]
- [[concepts/serving-performance-measurement]]
- [[hardware/aws-efa]]
- [[comparisons/scaling-1-to-5-machines]]

## Sources

- [[sources/disaggregated-serving-papers-2026-05]]
- [[sources/nvidia-dynamo-multinode-2026-05]]
