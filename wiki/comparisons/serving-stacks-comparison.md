---
tags: [comparison, infrastructure, serving]
last_updated: 2026-05-08
source_count: 2
---

# Serving stacks comparison — when to pick what

The cross-cutting comparison matrix for every credible open-source LLM
serving stack as of May 2026. The wiki's primary baseline remains
[[infrastructure/vllm]] + [[infrastructure/nvidia-dynamo]] (multi-node);
this page lists the alternatives and concrete situations in which they win.

For the master menu of pages, see [[infrastructure/serving-stack-landscape]].
For the multi-node-scaling decision tree, see
[[comparisons/scaling-1-to-5-machines]].

## Headline matrix

[Source: [[sources/serving-stacks-alternatives-2026-05]]]

| Stack | License | Distinguishing strength | Tool calling | Multi-node | A10G fit |
|---|---|---|---|---|---|
| [[infrastructure/vllm]] | Apache-2.0 | Reference impl; broad parser surface; V1 engine | many model-aware parsers | yes | ✅ |
| [[infrastructure/sglang]] | Apache-2.0 | RadixAttention prefix reuse; CFSM JSON | 16+ parsers | yes | ✅ |
| [[infrastructure/tensorrt-llm]] | Apache-2.0 | NVFP4/MXFP4 native; peak NVIDIA | limited; via Dynamo | yes | ✅ |
| [[infrastructure/lmdeploy]] | Apache-2.0 | AWQ-W4A16 + KV-quant + prefix-cache simultaneously | InternLM2.5/Llama3.1 | yes | ✅✅ |
| [[infrastructure/tgi]] | Apache-2.0 | ⚠ **Maintenance mode** | grammar Guidance | limited | ✅ |
| [[infrastructure/nvidia-dynamo]] | Apache-2.0 | Orchestrator + PD-disagg + KV-aware routing | inherits backend | yes | overkill 1-GPU |
| [[infrastructure/triton-vs-dynamo|Dynamo-Triton]] | BSD-3 | Multi-framework ML | n/a | yes | general ML |
| [[infrastructure/ray-serve-llm]] | Apache-2.0 | Multi-LoRA + multi-model + autoscale | inherits vLLM | yes | ✅ |
| [[infrastructure/vllm-production-stack]] | Apache-2.0 | Helm + KV-aware router + LMCache | inherits vLLM | yes | ✅ |
| [[infrastructure/llm-d]] | Apache-2.0 | KServe operator + Inference Gateway | inherits vLLM | yes | ✅ |
| [[infrastructure/aibrix]] | Apache-2.0 | Heterogeneous-GPU control plane | inherits engine | yes | ✅ |
| [[infrastructure/leaderworkerset]] | Apache-2.0 | K8s API for multi-host TP+PP replicas | n/a | yes | ⚠ rare on g5 |
| [[infrastructure/serving-stack-landscape\|DeepSpeed-FastGen]] | Apache-2.0 | Dynamic SplitFuse (slowing) | none | limited | ⚠ |
| [[infrastructure/serving-stack-landscape\|Friendli Engine]] | Closed | Patent of iteration batching | function calling | yes | commercial |
| [[infrastructure/serving-stack-landscape\|Aphrodite]] | AGPL-3.0 | Widest quantization matrix | inherits vLLM | limited | ✅ |
| [[infrastructure/serving-stack-landscape\|llama.cpp / Ollama]] | MIT | CPU/edge/Apple Silicon | llama-server FC | no | ❌ |

## Decision matrix (one-line)

| If you need... | Pick |
|---|---|
| Default open-source serving, broad model support | [[infrastructure/vllm]] |
| Prefix-heavy / agentic / RAG / multi-turn | [[infrastructure/sglang]] |
| Max throughput on A10G via AWQ INT4 | [[infrastructure/lmdeploy]] (limited tool calling) or [[infrastructure/vllm]] |
| Peak NVIDIA performance on H100/H200/B200 | [[infrastructure/tensorrt-llm]] under [[infrastructure/nvidia-dynamo]] |
| Multi-replica vLLM cluster on K8s with prefix routing | [[infrastructure/vllm-production-stack]] |
| Full K8s operator (canary, multi-model, predictive+generative) | [[infrastructure/llm-d]] (with KServe) |
| Multi-host single very large model (TP×PP across nodes) | [[infrastructure/leaderworkerset]] |
| Multi-LoRA + heterogeneous GPU pool + autoscale | [[infrastructure/ray-serve-llm]] or [[infrastructure/aibrix]] |
| Disaggregated prefill/decode | [[infrastructure/nvidia-dynamo]] |
| Quantization format vLLM doesn't ship (BitNet, AQLM, QuIP#, ExLlamaV3) | Aphrodite (AGPL!) |
| CPU / Apple Silicon / edge | llama.cpp / Ollama (out of scope) |

## Workload-driven recommendations

### Workload: tool-calling agentic, single g5.xlarge baseline

Best fit: **[[infrastructure/vllm]] + [[infrastructure/sglang]]** as a
binary choice (run benchmarks against your own workload to pick).
vLLM has the broadest parser surface for the wiki's candidate models
(Qwen3-Coder, Devstral, GLM-4.5, gpt-oss, Kimi-K2). SGLang's RadixAttention
materially outperforms on multi-turn agentic traffic.

Avoid LMDeploy here despite the throughput win — its tool-calling parser
list is too narrow.

### Workload: 32B+ model, fitting tight on a 24 GB A10G

Best fit: **[[infrastructure/lmdeploy]] AWQ-W4A16 + KV INT8 + prefix cache
simultaneously**. The combination most engines disallow, LMDeploy supports.
Accept the tool-calling parser limitation if your workload is mostly code
generation, not multi-turn tool use.

### Workload: scaling 1 → 2 → 5 g5.xlarge replicas

Best fit: **[[infrastructure/vllm]] replicas + [[infrastructure/vllm-production-stack]]**
with prefix-aware routing. EFA is unavailable on g5; this is purely
multi-replica DP, not multi-node TP. See
[[comparisons/scaling-1-to-5-machines]].

### Workload: multi-node single model (Llama-3.1-405B / Kimi-K2)

Best fit: **[[infrastructure/vllm]] or [[infrastructure/sglang]] under
[[infrastructure/leaderworkerset]] on EKS** with EFA-capable instances
(p5/p5e/p5en/p6). Apply the canonical vLLM rule: TP=GPUs-per-node,
PP=nodes.

### Workload: NVIDIA-peak performance, NVFP4 native

Best fit: **[[infrastructure/tensorrt-llm]] backend under
[[infrastructure/nvidia-dynamo]]** on p6-b200 / p6e-gb200. Dynamo provides
the OpenAI tool-calling layer; TRT-LLM provides the kernel-peak performance.

### Workload: heterogeneous prefill (H100) + decode (L40S) split

Best fit: **[[infrastructure/nvidia-dynamo]] with disaggregated serving** —
the productionization of the Splitwise paper. Plan for cluster placement
group + EFA across the prefill and decode pools.

## A10G g5.xlarge — what changes for this wiki

The wiki's primary baseline is **g5.xlarge + vLLM + (optionally) Dynamo**.
What does this expansion change?

1. **Dynamo's value on a single g5.xlarge is still nil.** Confirmed by
   Dynamo's own README. Don't deploy Dynamo unless multi-node or
   multi-replica.
2. **vLLM remains the default engine.** SGLang is a serious alternative for
   prefix-heavy workloads but doesn't replace vLLM as default — verify with
   your workload.
3. **For 1 → 2 → 5 g5 scaling**: use
   [[infrastructure/vllm-production-stack]] as the canonical answer.
4. **TGI is deprecated.** If old wiki content cites TGI, treat it as legacy.
5. **LMDeploy is undervalued for AWQ-INT4 throughput** on A10G — but only
   for non-tool-calling workloads.
6. **Triton was renamed Dynamo-Triton.** Disambiguate in any older content.
   See [[infrastructure/triton-vs-dynamo]].

## Related

- [[infrastructure/serving-stack-landscape]] — master menu
- [[comparisons/scaling-1-to-5-machines]] — multi-machine recipes
- [[concepts/parallelism-strategies]]
- [[concepts/disaggregated-serving]]
- [[concepts/serving-performance-measurement]]
- [[hardware/aws-efa]]

## Sources

- [[sources/serving-stacks-alternatives-2026-05]]
- [[sources/sglang-distributed-2026-05]]
