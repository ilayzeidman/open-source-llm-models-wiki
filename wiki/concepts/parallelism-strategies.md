---
tags: [concepts, parallelism, multi-gpu, multi-node]
last_updated: 2026-05-08
source_count: 3
---

# Parallelism strategies — TP, PP, DP, EP

The four axes by which an LLM serving job can be split across more GPUs.
Picking the right combination depends on (a) model size relative to a single
GPU's VRAM, (b) interconnect speed (NVLink / PCIe / EFA / ENA), (c) target
latency vs throughput, and (d) MoE vs dense.

## The four axes

| Strategy | What it splits | Communication pattern | Best when |
|---|---|---|---|
| **Tensor parallel (TP)** | Each layer's matmul across N GPUs | AllReduce per layer | Model > 1 GPU; fast intra-node interconnect (NVLink / NVSwitch). Lowest TTFT/ITL per request. |
| **Pipeline parallel (PP)** | Layers across N stages | P2P between adjacent stages only | Cross-node, or PCIe/L40S/A10G (no NVLink). Far less comm volume than TP. |
| **Data parallel (DP)** | Independent replicas | None for dense; AllReduce-coordinated dummy passes for MoE | Throughput; one replica fits a GPU/node. Embarrassingly parallel for dense. |
| **Expert parallel (EP)** | MoE experts across GPUs | All-to-all (DeepEP / PPLX kernels) | Sparse MoE only. `EP_SIZE = TP_SIZE × DP_SIZE`. |

## Communication volumes (the rule of thumb)

For per-step communication on a transformer of L layers, hidden size H,
sequence length S, batch B:

| Strategy | Comm volume |
|---|---|
| TP (per-layer AllReduce) | `4 · B · S · H · L · bytes` |
| PP (per-step adjacent-stage P2P) | `B · S · H · (P − 1) · bytes` |
| DP (dense) | 0 |
| EP (per-token all-to-all) | proportional to active experts × top-k |

**PP transfer is roughly `4·L`× lower than TP.** That's why PP is the right
choice across slow links (no NVLink, no EFA), even though TP gives lower
per-token latency on fast links.

[Source: [[sources/sglang-distributed-2026-05]] — LMSYS chunked-pipeline blog
quantifies "near order-of-magnitude reduction"]

## vLLM's canonical rule

[Source: [[sources/vllm-distributed-serving-2026-05]]]

> "**The tensor parallel size should be the number of GPUs in each node, and
> the pipeline parallel size should be the number of nodes.**"

Example: 16 GPUs across 2 nodes → `--tensor-parallel-size 8 --pipeline-parallel-size 2`.

For uneven GPU counts within a node: "Enable pipeline parallelism, which
splits the model along layers and supports uneven splits."

For nodes without NVLink (g5/g6/g6e/L40S on AWS): "leverage pipeline
parallelism instead of tensor parallelism for higher throughput and lower
communication overhead."

## When to use which

| Goal / Constraint | Scale by | Why |
|---|---|---|
| Model fits one GPU/node, need more QPS | **DP replicas** | Zero inter-node traffic; embarrassingly parallel; survives partial failures |
| Model > 1 GPU but fits one node | **TP within node** | NVLink/NVSwitch tolerates AllReduce-per-layer; lowest TTFT/ITL |
| Model > 1 node | **TP within node + PP across nodes** | Official vLLM rule |
| Tight TTFT for single user | **TP** (more, better) | Splits compute, reduces wall-clock per token |
| Maximize throughput at high concurrency | **DP replicas** (or DP+EP for MoE) | Independent batches; no sync tax |
| Disaggregate prefill from decode | **Disaggregated serving** | Each phase scales independently with right hardware |
| **Slow inter-node link** (no NVLink, no EFA) — i.e. **g5 on AWS** | **PP across nodes**, replicas, or DP — **avoid TP across nodes** | TP comm scales with `4·L` |
| MoE at scale (DeepSeek-class) | **DP attention + EP MoE** with `--enable-expert-parallel` | Avoid 8× MLA KV duplication |

## MoE-specific rules

[Source: [[sources/vllm-distributed-serving-2026-05]] — vLLM EP docs + ROCm
Playbook]

- DeepSeek with MLA: "**Always use EP=1 with DP to avoid 8× KV cache
  duplication.**"
- Crossover (MoE workloads, ROCm benchmarks): ≤128 concurrent → TP+EP wins;
  ≥512 → DP+EP wins. Crossover band 256–512.
- All-to-all backend choice:
  - `deepep_high_throughput` (grouped GEMM) — prefill-heavy
  - `deepep_low_latency` (CUDA graphs) — decode-heavy
  - `flashinfer_nvlink_*` — cross-node NVLink (NVL72-class)

## SGLang's chunked pipeline parallelism

[Source: [[sources/sglang-distributed-2026-05]] — LMSYS Jan 2026]

SGLang's PP combines three techniques to push PP scaling further than
vLLM's current implementation:

1. **Chunked Pipeline Parallelism (CPP)** — partition prompts into 4–6 K
   token chunks; stage 1 advances to chunk 2 while stage 2 processes chunk 1.
2. **Asynchronous P2P Communication** — `P2PWork` handle returns without
   GPU-side blocking.
3. **Dynamic Chunking** — model "the cumulative runtime as a quadratic
   function of sequence length" to keep per-chunk times equal.

Result on DeepSeek-V3.1: "**PP4 TP8 yields 3.31× Prefill Throughput**" vs
TP8; outperforms TP32 by 30.5%. Qwen3-235B-A22B-FP8 PP8: TTFT 55.5 s →
10.5 s.

This is the strongest 2026 evidence that PP > TP on slow links at scale.

## Multi-replica vs multi-node TP/PP — decision rubric

The most common scaling question on the wiki's primary baseline (g5.xlarge):
"if I have 2 g5.xlarge boxes, do I run them as 2 TP replicas or 2
independent DP replicas?"

For **g5.xlarge** specifically (no EFA, no NVLink across instances), the
answer is unambiguous: **DP replicas behind a router**. TP across two
g5.xlarge instances would route AllReduce traffic over ENA at ≤25 Gbps —
catastrophic for any nontrivial model.

For larger boxes that do support EFA (g6e, p4d, p5, p6), TP across 2 nodes
becomes viable, and the canonical vLLM rule (TP=GPUs-per-node, PP=nodes)
applies. See [[hardware/aws-efa]].

## A10G g5 — what's actually viable

| Pattern | Viable on g5? | Comment |
|---|---|---|
| 1 replica per g5.xlarge, multiple g5.xlarge behind router (DP across instances) | **✅ canonical** | The wiki's "scale 1 → 2 → 5 g5" answer. See [[comparisons/scaling-1-to-5-machines]]. |
| TP=2 within g5.12xlarge / g5.48xlarge | ✅ | PCIe Gen4; works but adds comm overhead. |
| TP=4 within g5.12xlarge | ✅ | Standard for 70B AWQ. |
| TP=8 within g5.48xlarge | ✅ | Standard for 70B FP16. |
| **TP across two g5.xlarge instances** | ❌ | No EFA, no NVLink — TP comm saturates ENA. |
| **PP across two g5.xlarge instances** | ⚠ marginal | PP comm tolerates ENA, but rare scenario — most multi-instance models on g5 fit in a single g5 at AWQ. |
| EP across multiple g5 instances | ❌ effectively | All-to-all without EFA is infeasible at production scale. |

## Related

- [[infrastructure/vllm]]
- [[infrastructure/sglang]]
- [[infrastructure/leaderworkerset]] — K8s API for multi-host TP+PP
- [[hardware/aws-efa]] — what's required for cross-node TP
- [[hardware/multi-gpu-options]]
- [[concepts/disaggregated-serving]]
- [[comparisons/scaling-1-to-5-machines]]

## Sources

- [[sources/vllm-distributed-serving-2026-05]]
- [[sources/sglang-distributed-2026-05]]
- [[sources/aws-efa-multinode-2026-05]]
