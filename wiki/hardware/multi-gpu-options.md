---
tags: [hardware, gpu, aws, multi-gpu, multi-node]
last_updated: 2026-05-08
source_count: 4
---

# Multi-GPU options on AWS

When a model doesn't fit on the largest single-GPU AWS SKU (now **L40S 48 GB**,
not A10G 24 GB — see [[hardware/g6e-l40s]]), step up. This page covers the
multi-GPU decision logic; for the full price/spec menu including all
single-GPU options, see [[hardware/aws-gpu-landscape]]. For multi-**node**
networking specifics, see [[hardware/aws-efa]]. For the parallelism axes
themselves (TP / PP / DP / EP), see [[concepts/parallelism-strategies]].

## Updated decision tree

The "single GPU on AWS" universe now has 5 SKUs (g4dn / g5 / g6 / g6e / no
single-card P-class). Updated:

1. **Fits on 1× L40S 48 GB at FP16/FP8 with usable ctx?** → g6e.xlarge ($1.861)
2. **Fits on 1× A10G 24 GB at AWQ INT4?** → g5.xlarge ($1.006) — original baseline
3. **Need FP8 native at 24 GB?** → g6.xlarge ($0.80) on L4 (lower bandwidth — verify decode tps)
4. **Need 96 GB on single node?** → g5.12xlarge / g6.12xlarge / g6e.12xlarge (4×24/4×48 PCIe)
5. **Need NVLink for TP?** → p4d/p4de/p5/p6 only (not G-class)
6. **Need 1 TB+ for frontier MoE?** → p5e/p5en (H200) or p6-b200

## When does multi-GPU actually help?

[Source: [[sources/vllm-distributed-serving-2026-05]]]

- **Tensor parallelism (TP)** splits model weights across N GPUs. Required when
  weights don't fit a single card. Per-GPU compute drops linearly but communication
  overhead eats some of it (notable on PCIe-only g5/g6/g6e; minor on NVLink p4d/p5/p6).
- **Pipeline parallelism (PP)** stages layers across GPUs. Better for very large
  models, worse for latency. Per-step transfer volume is `B·S·H·(P-1)·bytes` —
  near order-of-magnitude lower than TP — making PP the right choice across
  slow links.
- **Data parallelism (DP)** runs N independent replicas. Zero inter-node
  comm for dense models; embarrassingly parallel for throughput.
- **Expert parallelism (EP)** for MoE — vLLM `--enable-expert-parallel`. Required for
  large MoEs like Qwen3-Coder-480B and DeepSeek-V3.1.
- **Disaggregated prefill/decode** (see [[concepts/disaggregated-serving]] and
  [[infrastructure/nvidia-dynamo]]) splits the prefill workload from decode
  across separate GPU groups — only meaningful when you have ≥2 GPUs and a
  streaming workload with mixed prompt lengths.

The full TP / PP / DP / EP decision rubric is at
[[concepts/parallelism-strategies]].

## Multi-node vs multi-replica

A fundamentally different question from "more GPUs in one node":

- **Multi-replica (DP across nodes)**: each node runs an independent vLLM
  replica; a router (KV-aware) load-balances across them. **No inter-node
  TP traffic.** This is the answer for "I have a model that fits a single
  node, I want more QPS." See [[comparisons/scaling-1-to-5-machines]].
- **Multi-node TP+PP**: one model spans multiple nodes (e.g. Llama-3.1-405B
  across 2× p5.48xlarge with TP=8, PP=2). **Requires fast inter-node
  networking** — EFA on AWS. See [[hardware/aws-efa]] and
  [[infrastructure/leaderworkerset]].

⚠ **g5 lacks EFA**, so multi-node TP across g5 instances is non-viable.
g5 multi-machine scaling = multi-replica only.

## Model-size → smallest single-node AWS box

[Source: [[sources/aws-extended-gpu-pricing-2026-05]]]

| Model class | Best single-node AWS box | Smallest fit |
|---|---|---|
| ≤ 8B dense (e.g. Granite, Hermes-3-8B) | g4dn.xlarge ($0.526) | AWQ INT4 |
| 14B dense (Qwen3-14B, Phi-4-14B) | g5.xlarge ($1.006) | AWQ INT4 |
| 24B dense (Devstral-Small, Mistral-Small-3.2) | g5.xlarge ($1.006) | AWQ INT4 |
| 24B dense FP16 | g6e.xlarge ($1.861) | FP16 |
| 30B-A3B MoE (Qwen3-Coder) | g5.xlarge ($1.006) AWQ INT4 / g6e.xlarge FP8 | both fit |
| 32B dense (Qwen3-32B, Qwen2.5-Coder-32B) | g6e.xlarge ($1.861) | AWQ INT4 generous ctx |
| 70B dense (Llama-3.3-70B, Hermes-4-70B) | g5.48xlarge ($16.29) or g6e.12xlarge ($10.49) | AWQ INT4 TP=4–8 |
| 106B/12B MoE (GLM-4.5-Air) | g6e.12xlarge ($10.49) | FP8 TP=4 (TP=2 doesn't fit) |
| 109B/17B MoE (Llama-4-Scout) | g6e.48xlarge ($30.13) | INT4 TP=8 |
| 123B dense (Devstral-2-123B) | g6e.12xlarge ($10.49) | AWQ INT4 TP=4 |
| 355B/32B MoE (GLM-4.5) | p4d.24xlarge ($32.77) | AWQ INT4 TP=8 |
| 480B/35B MoE (Qwen3-Coder-480B) | p5e/p5en (H200) | FP8 TP=8 |
| 671B/37B MoE (DeepSeek-V3.1) | p5e/p5en (H200) | FP8 TP=8 |
| 1T/32B MoE (Kimi-K2) | p5e/p5en or p6-b200 | FP8 TP=8 |

See [[comparisons/models-by-budget]] for the budget-first cut of this same data.

## Cost comparison: same 32B model, different boxes

For a 32B AWQ-INT4 model (≈19 GB weights), e.g. [[models/qwen2.5-coder-32b]]:

| Box | $/hr | $/month (24×7) | Notes |
|---|---:|---:|---|
| g5.xlarge | $1.006 | ~$734 | tight ctx (~7k) |
| g6.xlarge | $0.8048 | ~$587 | 24 GB L4, lower memory bw |
| **g6e.xlarge** | **$1.861** | **~$1,358** | **comfortable 32B AWQ INT4 with 80K ctx** |
| g5.12xlarge (TP=4 FP16) | $5.672 | ~$4,140 | full FP16, much higher quality |
| g6e.12xlarge (TP=4 FP8/FP16) | $10.493 | ~$7,660 | full FP8, 192 GB headroom |

For most users the new sweet spot is **g6e.xlarge** rather than the older g5.12xlarge
upgrade — same budget tier (~$1,400/mo), single-GPU simplicity, full 32B AWQ context.

Spot pricing typically 30–70% off on-demand and is the realistic deployment for
non-prod workloads.

## Related

- [[hardware/aws-gpu-landscape]] — full price/spec menu
- [[hardware/a10g-g5xlarge]], [[hardware/g6e-l40s]] — per-GPU deep dives
- [[hardware/aws-efa]] — multi-node networking specifics
- [[concepts/parallelism-strategies]] — TP/PP/DP/EP decision rubric
- [[infrastructure/quantization]]
- [[infrastructure/nvidia-dynamo]]
- [[infrastructure/leaderworkerset]] — K8s API for multi-host TP+PP
- [[comparisons/scaling-1-to-5-machines]] — multi-replica recipe

## Sources

- [[sources/aws-extended-gpu-pricing-2026-05]]
- [[sources/aws-ec2-pricing-2026-05]]
- [[sources/vllm-distributed-serving-2026-05]]
- [[sources/aws-efa-multinode-2026-05]]
