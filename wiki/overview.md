---
tags: [overview]
last_updated: 2026-05-08
source_count: 26
---

# Overview

The research question: **which open-source LLMs are credible candidates for tool
selection and complex code generation, and which AWS GPU instance hits the right
quality / cost / fit balance for each?**

The original wiki was scoped to a single hardware target (g5.xlarge / A10G 24 GB).
That target is preserved as the **baseline** — but the wiki now covers the full
AWS GPU spectrum from g4dn ($0.526/hr) to p6-b200 ($113.93/hr) and the matching
open-source model tiers from 7B to 1T parameters. See
[[hardware/aws-gpu-landscape]] and [[comparisons/models-by-budget]].

The wiki also now covers **alternative serving stacks** beyond the original
vLLM + NVIDIA Dynamo baseline (SGLang, TensorRT-LLM, LMDeploy, Ray Serve LLM,
vLLM Production Stack, llm-d, AIBrix, LeaderWorkerSet) and **multi-machine
scaling** (1 → 2 → 5 g5.xlarge replicas) plus **multi-node serving performance
measurement methodology** (TTFT/ITL/goodput, GenAI-Perf, MLPerf v5.0). See
[[infrastructure/serving-stack-landscape]] and
[[comparisons/scaling-1-to-5-machines]].

## Hard constraints (still)

- **Capability focus**: [[concepts/tool-selection]] and [[concepts/code-generation]].
  Pure conversational quality is secondary.
- **License**: commercial-friendly preferred. CC-BY-NC-4.0 (xLAM) and MNPL
  (Codestral 22B) are non-commercial blockers; Llama 4 Community is OK below
  700M MAU.

## Soft constraints (relaxed)

- **Serving stack**: [[infrastructure/vllm]] (latest stable: 0.20.x) is the
  default. [[infrastructure/nvidia-dynamo]] is the canonical multi-node
  orchestrator; Dynamo's value is multi-GPU/multi-node, on a single GPU use
  vLLM directly per Dynamo's own README. The wiki now also documents
  [[infrastructure/sglang]] (RadixAttention, prefix-heavy/agentic workloads),
  [[infrastructure/tensorrt-llm]] (NVIDIA peak / NVFP4),
  [[infrastructure/lmdeploy]] (AWQ-W4A16 specialty), and
  [[infrastructure/ray-serve-llm]] / [[infrastructure/vllm-production-stack]] /
  [[infrastructure/llm-d]] / [[infrastructure/aibrix]] for K8s orchestration.
  See [[infrastructure/serving-stack-landscape]] for the master menu.

- **GPU**: now spans the full AWS NVIDIA lineup — T4 (sm_75), A10G (sm_86), L4
  (sm_89), L40S (sm_89), A100 (sm_80), H100 (sm_90), H200 (sm_90), B200 (sm_100).
- **Instance**: g5.xlarge remains the **baseline**, but **g6e.xlarge ($1.861/hr,
  L40S 48 GB)** is now the recommended sweet spot for 24–32B models that want
  full quality without multi-GPU comms overhead.
- **Multi-machine serving**: g5 is **EFA-less**; cross-node TP is non-viable.
  Scale 1 → 2 → 5 g5 by **independent replicas behind a KV-aware router**, not
  multi-node TP. EFA is available on g6e and above. See
  [[hardware/aws-efa]] and [[comparisons/scaling-1-to-5-machines]].

## What "wins" looks like (by budget tier)

[Source: [[sources/aws-extended-gpu-pricing-2026-05]], [[comparisons/models-by-budget]]]

| Tier | $/hr | Best tool calling | Best complex code | Best agentic SWE-bench |
|---|---:|---|---|---|
| < $1/hr (T4/L4) | $0.53–$0.80 | [[models/granite-3.1-8b\|Granite-3.3-8B]] | [[models/qwen2.5-coder-7b]] | [[models/devstral-small]] AWQ |
| $1–$2/hr (A10G **baseline**) | $1.006 | [[models/granite-3.1-8b\|Granite-3.3-8B]] | [[models/qwen3-coder-30b-a3b]] AWQ | [[models/devstral-small]] (SWE-V 53.6) |
| $2–$5/hr (L40S 48GB) | $1.861–$3.0 | [[models/qwen3-coder-30b-a3b]] FP8 | [[models/qwen3-32b]] / [[models/devstral-small]] FP16 | [[models/devstral-small]]-2 (SWE-V 68) |
| $5–$15/hr (PCIe TP) | $5.67–$10.5 | [[models/glm-4.5-air]] FP8 TP=4 | [[models/qwen2.5-coder-32b]] FP16 TP=4 | [[models/glm-4.5-air]] |
| $15–$33/hr (8× G or NVLink) | $16.29–$32.77 | [[models/llama-3.3-70b-instruct]] | [[models/devstral-small]]-2-123B | [[models/qwen3-coder-480b]] INT4 |
| $33–$115/hr (P5/P6) | $40.97–$113.93 | [[models/qwen3-coder-480b]] | [[models/qwen3-coder-480b]] / [[models/kimi-k2]] | [[models/kimi-k2]] K2.6 (**SWE-V 80.2**) |

## Recommended defaults — by use case

For commercial deployment, May 2026:

| Use case | Recommendation | Box | Why |
|---|---|---|---|
| Tool calling, cheapest | [[models/granite-3.1-8b\|Granite-3.3-8B]] AWQ | g4dn.xlarge / g5.xlarge | Apache-2.0, `granite` parser, HE 89.73 |
| Code+tools, A10G baseline | [[models/qwen3-coder-30b-a3b]] AWQ | g5.xlarge | Apache-2.0, `qwen3_coder` parser, MoE 3.3B active |
| Agent SWE-bench, A10G | [[models/devstral-small]] AWQ | g5.xlarge | Apache-2.0, SWE-V 53.6→68 |
| Code+tools, full quality | [[models/qwen3-coder-30b-a3b]] FP8 | **g6e.xlarge** | 256K ctx, single-GPU |
| 70B generalist | [[models/llama-3.3-70b-instruct]] AWQ | g6e.12xlarge | TP=4 PCIe, 192 GB |
| Frontier open-source code+tools | [[models/qwen3-coder-480b]] / [[models/kimi-k2]] | p5e/p5en or p6-b200 | only multi-node options |
| Best Apache-2.0 reasoning | [[models/gpt-oss-20b]] (high reasoning) | p5 slice or g6e.xlarge BF16 | SWE-V 60.7 small footprint |

See [[comparisons/tool-calling-models-on-a10g]] for the original A10G master table
and [[comparisons/models-by-budget]] for the broader budget cut.

## Candidate landscape (May 2026)

[Source: [[sources/bfcl-leaderboard-2026-05]], [[sources/code-benchmarks-2026-05]],
[[sources/qwen3-coder-cards]], [[sources/deepseek-v3-r1-family]], [[sources/glm-4.5-cards]],
[[sources/kimi-k2-cards]], [[sources/gpt-oss-cards]], [[sources/devstral-small-cards]],
[[sources/llama-4-cards]]]

### A10G-class (≤ 24 GB single GPU)

- **Tool-calling specialists**: [[models/hermes-3-llama-3.1-8b]] (Llama community,
  mainline parser), [[models/xlam-7b]] (CC-BY-NC, research only),
  [[models/functionary-small]] (MIT, no mainline parser).
- **Code specialists**: [[models/qwen2.5-coder-7b]], [[models/qwen2.5-coder-14b]],
  [[models/qwen2.5-coder-32b]] (Apache-2.0, parser flaky); **[[models/qwen3-coder-30b-a3b]]
  (Apache-2.0, new `qwen3_coder` parser — supersedes the Qwen2.5-Coder line)**;
  [[models/codestral-22b]] (MNPL — non-commercial); [[models/deepseek-coder-v2-lite]]
  (DeepSeek License — OK); **[[models/devstral-small]] (Apache-2.0, SWE-V 53–68%)**.
- **Generalists**: [[models/llama-3.1-8b-instruct]], [[models/mistral-small-24b]],
  [[models/granite-3.1-8b]], [[models/phi-3-medium-14b]] (no parser — disqualified),
  **[[models/qwen3-32b]] (Apache-2.0, hybrid reasoning)**.

### L40S-class (48 GB single GPU on g6e)

Adds: full Qwen3-Coder-30B-A3B at FP8 + 256K ctx, full Devstral-Small-24B at FP16,
Mistral-Small-3.2-24B at FP16 with KV headroom, and any 32B AWQ INT4 with 80K ctx.

### Multi-GPU PCIe (g6e.12xlarge / g5.48xlarge)

Adds: [[models/llama-3.3-70b-instruct]], **[[models/glm-4.5-air]]** (106B/12B MoE,
MIT; parent GLM-4.5 SWE-V 64.2 / TAU 70.1), Devstral-2-123B, Hermes-4-70B, Llama-4-Scout INT4.

### NVLink / multi-node (p4d / p5 / p5e / p6)

Adds the open-source frontier: **[[models/deepseek-v3.1]]** (671B/37B, SWE-V 66.0),
**[[models/qwen3-coder-480b]]** (480B/35B, SWE-V 66.5), **[[models/kimi-k2]]**
(1T/32B, K2.6 SWE-V **80.2**), [[models/llama-4-scout]]/Maverick, GLM-4.5 (355B),
**[[models/gpt-oss-120b]]** (MXFP4 native, requires sm_90+).

## Pitfalls and caveats (May 2026)

1. **Qwen2.5-Coder is superseded by Qwen3-Coder** for new deployments. The new
   `qwen3_coder` vLLM parser fixes the flaky-`hermes` issue documented in earlier
   wiki entries.
2. **Codestral 22B is superseded by Devstral-Small** for commercial use — same
   Mistral lineage, **Apache-2.0** instead of MNPL.
3. **MXFP4 weights** ([[models/gpt-oss-20b]] / [[models/gpt-oss-120b]]) require Hopper or newer (sm_90+).
   On A10G/L4/L40S, vLLM dequantizes to BF16 (≈ 2× VRAM). Practical native AWS
   target = p5+ / p6-b200.
4. **AWS does not sell single-GPU H100/H200/B200 SKUs** — minimum is the 8×
   chassis. Plan for multi-tenancy or Capacity Blocks at that tier.
5. **Kimi-K2 does NOT fit 8× H100 80GB at FP8** — needs H200/B200 single-node.
   p5.48xlarge is too small.
6. **Llama 4 Community License** allows commercial use only below 700M MAU; not OSI.
7. **g6e.xlarge ($1.861/hr) is the cheapest 48 GB single-GPU box on AWS** — the most
   under-used SKU for 24–32B model serving. Often a better choice than g5.12xlarge
   ($5.67/hr) for a single deployment.
8. **L4 (g6) lower memory bandwidth than A10G**: 300 vs 600 GB/s. Despite newer
   architecture and FP8, L4 typically loses to A10G on decode throughput.
9. **All public BFCL/SWE-bench/LCB scores are FP16/BF16/FP8** — quantized AWQ INT4
   evals essentially absent; quality drop unmeasured for most candidates.
10. **NVIDIA Dynamo offers essentially nothing for a single GPU**. Multi-node value;
    on g5.xlarge / g6e.xlarge use vLLM directly.

## Open questions still worth investigating

- **Real measured AWQ INT4 throughput on A10G / L4 / L40S** for each candidate —
  no public per-model numbers.
- **Devstral-Small-2-2512 and Kimi-K2.6 BFCL v3/v4** — neither is published yet.
- **gpt-oss community AWQ INT4 quality** on A10G — does the quant preserve SWE-bench scores?
- **τ-bench** scores for the new candidates ([[models/qwen3-coder-30b-a3b]],
  [[models/devstral-small]], [[models/glm-4.5-air]]) — closer to real agentic
  performance than BFCL.
- **GLM-4.5-Air vs GLM-4.6** — Z.ai released 4.6; impact on Air-tier model not
  yet documented.
- **vLLM vs SGLang on A10G for agent workloads** — published comparisons all
  use H100. Need a g5.xlarge-specific evaluation for tool-calling agentic
  traffic (where SGLang's RadixAttention should win materially).
- **No widely-adopted public agentic-tool-call serving trace.** Production
  benchmarks against agentic workloads currently require capturing your own
  traces and replaying. BFCL v3/v4, AgentBoard, MCP-Bench, MINT exist but
  are synthetic. See [[concepts/serving-performance-measurement]].

## Scaling and serving-stack expansion (May 2026)

Three new threads, each with its own decision tree:

| Thread | Entry point |
|---|---|
| **Alternative serving stacks** (when vLLM isn't the right choice) | [[infrastructure/serving-stack-landscape]], [[comparisons/serving-stacks-comparison]] |
| **Scaling 1 → 2 → 5 g5.xlarge** (independent replicas + router) | [[comparisons/scaling-1-to-5-machines]] |
| **Multi-node serving performance measurement** (goodput, MLPerf, GenAI-Perf) | [[concepts/serving-performance-measurement]] |

## Sources

- [[sources/aws-ec2-pricing-2026-05]]
- [[sources/aws-extended-gpu-pricing-2026-05]]
- [[sources/nvidia-a10g-specs]]
- [[sources/nvidia-l4-l40s-specs]]
- [[sources/nvidia-dynamo-readme]]
- [[sources/vllm-tool-calling-docs]]
- [[sources/vllm-quantization-docs]]
- [[sources/bfcl-leaderboard-2026-05]]
- [[sources/code-benchmarks-2026-05]]
- [[sources/qwen3-coder-cards]]
- [[sources/qwen3-dense-cards]]
- [[sources/deepseek-v3-r1-family]]
- [[sources/llama-4-cards]]
- [[sources/glm-4.5-cards]]
- [[sources/kimi-k2-cards]]
- [[sources/gpt-oss-cards]]
- [[sources/devstral-small-cards]]
- [[sources/serving-stacks-alternatives-2026-05]]
- [[sources/vllm-distributed-serving-2026-05]]
- [[sources/nvidia-dynamo-multinode-2026-05]]
- [[sources/disaggregated-serving-papers-2026-05]]
- [[sources/k8s-llm-orchestration-2026-05]]
- [[sources/aws-efa-multinode-2026-05]]
- [[sources/sglang-distributed-2026-05]]
- [[sources/llm-serving-perf-metrics-2026-05]]
- [[sources/llm-serving-perf-tools-2026-05]]
