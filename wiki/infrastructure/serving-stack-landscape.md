---
tags: [infrastructure, serving, master-table]
last_updated: 2026-05-08
source_count: 1
---

# Serving stack landscape — May 2026

The complete map of credible open-source LLM serving stacks that compete with
or complement [[infrastructure/vllm]] + [[infrastructure/nvidia-dynamo]]. Used
as the canonical entry point when evaluating "should we still use vLLM?" or
"is there a better fit for our workload?"

The wiki's primary deployment baseline (g5.xlarge / A10G / vLLM) is preserved.
This page exists because: (1) some workloads benefit materially from SGLang's
RadixAttention or LMDeploy's AWQ kernels; (2) Kubernetes-native production
clusters need orchestration above vLLM (Production Stack, llm-d, AIBrix,
Ray Serve LLM); (3) NVIDIA's own brand split (Triton → Dynamo-Triton; Dynamo
new) is easy to confuse — see [[infrastructure/triton-vs-dynamo]].

## Master comparison table

[Source: [[sources/serving-stacks-alternatives-2026-05]]]

| Stack | License | Primary value vs vLLM | Tool-calling | Multi-GPU | Multi-node | A10G fit | Dedicated page |
|---|---|---|---|---|---|---|---|
| **vLLM** | Apache-2.0 | Reference impl; broad parser surface | many model-aware | TP/PP/DP/EP | yes | ✅ | [[infrastructure/vllm]] |
| **SGLang** | Apache-2.0 | RadixAttention prefix reuse; CFSM JSON; "up to 6.4× higher throughput" on prefix-heavy | 16+ parsers | TP/PP/DP/EP | yes | ✅ | [[infrastructure/sglang]] |
| **TensorRT-LLM** | Apache-2.0 | Peak NVIDIA perf; NVFP4/MXFP4 native; tightly integrated with Dynamo | limited; via Dynamo | yes | yes | ✅ | [[infrastructure/tensorrt-llm]] |
| **LMDeploy** | Apache-2.0 | "1.8× vLLM" on InternLM; AWQ-W4A16 specialty | InternLM2.5 / Llama3.1 | yes | yes | ✅✅ | [[infrastructure/lmdeploy]] |
| **TGI** (Hugging Face) | Apache-2.0 | ⚠ **Maintenance mode** — HF recommends migrating to vLLM/SGLang | Guidance (grammar) | TP only | limited | ✅ | [[infrastructure/tgi]] |
| **NVIDIA Dynamo** | Apache-2.0 | Orchestrator above vLLM/SGLang/TRT-LLM; PD-disagg + KV-aware routing | inherits backend | yes | yes (designed) | overkill 1-GPU | [[infrastructure/nvidia-dynamo]] |
| **Dynamo-Triton** (formerly Triton) | BSD-3 | Multi-framework general inference (TF/ONNX/TRT/Python/FIL/...) | n/a | yes | yes | ✅ | [[infrastructure/triton-vs-dynamo]] |
| **Ray Serve LLM** | Apache-2.0 | Multi-model, multi-LoRA, autoscale orchestration over vLLM | inherits vLLM | yes | yes | ✅ | [[infrastructure/ray-serve-llm]] |
| **vLLM Production Stack** | Apache-2.0 | Helm-deployable vLLM cluster + LMCache + KV-aware router | inherits vLLM | yes | yes | ✅ | [[infrastructure/vllm-production-stack]] |
| **llm-d** | Apache-2.0 | K8s Operator + KServe Inference Gateway; precise prefix-aware routing | inherits vLLM | yes | yes | ✅ | [[infrastructure/llm-d]] |
| **AIBrix** | Apache-2.0 | Heterogeneous-GPU control plane; high-density LoRA; distributed KV cache | inherits engine | yes | yes | ✅ | [[infrastructure/aibrix]] |
| **LeaderWorkerSet** (LWS) | Apache-2.0 | K8s API for multi-host TP+PP replicas | n/a | yes | yes | ✅ | [[infrastructure/leaderworkerset]] |
| **DeepSpeed-FastGen / MII** | Apache-2.0 | Dynamic SplitFuse; "2.3× vLLM" (older bench); pace slowed | none documented | TP only | limited | ⚠ slowing | (brief below) |
| **Friendli Engine** | Closed | Patent-holder of iteration batching; "2-4× vLLM" (vendor) | function-calling API | yes | yes | commercial | (brief below) |
| **Aphrodite Engine** | AGPL-3.0 | Widest quantization matrix (BitNet, AQLM, QuIP#, ExLlamaV3, NVFP4...) | inherits vLLM | yes | limited | ✅ | (brief below) |
| **llama.cpp / Ollama** | MIT | CPU/edge/Apple Silicon; GGUF | llama-server FC | limited | no | ❌ not target | (brief below) |
| **KServe** | Apache-2.0 | K8s CRD platform unifying predictive + generative | per runtime | yes | yes | ✅ | (covered in [[infrastructure/llm-d]]) |

## Pick by workload

### "I want to keep vLLM but scale beyond a single GPU on K8s"
- 2 → 5 replicas, prefix-cache reuse: [[infrastructure/vllm-production-stack]]
- Full operator with canary, multi-model, scale-to-zero, predictive ML mixed in:
  [[infrastructure/llm-d]] (with KServe).
- Multi-host single model (e.g. Llama-3.1-405B across 2× 8×H100):
  [[infrastructure/leaderworkerset]] under [[infrastructure/vllm]] or
  [[infrastructure/sglang]].

### "Prefix-heavy / agentic / RAG / multi-turn workload"
[[infrastructure/sglang]]'s RadixAttention is the strongest match.
RadixAttention paper: "up to 6.4× higher throughput" on agent/RAG/multi-turn.
Use SGLang or vLLM Production Stack with **prefix-aware routing**.

### "I need to fit a 32B / 70B model on a single small GPU"
[[infrastructure/lmdeploy]] AWQ-W4A16 + KV-quant + prefix caching all
simultaneously. "1.8× vLLM throughput" reported on InternLM2-20B; "4-bit
2.4× FP16."

### "We need NVIDIA-peak performance and FP8/NVFP4 native"
[[infrastructure/tensorrt-llm]]; deploy under [[infrastructure/nvidia-dynamo]]
to get an OpenAI-compatible API + tool-calling.

### "Multi-LoRA, multi-model, autoscaling on heterogeneous GPUs"
[[infrastructure/ray-serve-llm]] or [[infrastructure/aibrix]].

### "Disaggregated prefill/decode at scale"
[[infrastructure/nvidia-dynamo]] (or vLLM PD-disagg directly, or SGLang
PD-disagg). See [[concepts/disaggregated-serving]] for the design rationale.

## Brief mentions (no dedicated page)

### DeepSpeed-FastGen / MII
Microsoft's iteration-batching engine. Distinguishing innovation: **Dynamic
SplitFuse** — "Long prompts are decomposed into smaller chunks across multiple
forward passes" and "Short prompts are fused to precisely fill a target token
budget." Reported wins vs vLLM (2024 benchmark): "up to 2.3× higher effective
throughput", "P95 generation-latency tail 3.7× smaller." **Status (May 2026):
latest release v0.3.3 dated March 25, 2025**; pace of releases has slowed.
Not a recommended new deploy. Tool calling: not documented.
Repo: https://github.com/deepspeedai/DeepSpeed-MII.

### Friendli Engine (FriendliAI)
The historical seed of the entire continuous-batching world.
Orca paper (OSDI 2022) introduced **iteration-level scheduling** and
**selective batching** — "36.9× throughput improvement at the same level of
latency" vs FasterTransformer on GPT-3 175B. Now industry standard
(underpins vLLM, SGLang, TGI, TRT-LLM, etc.). **Closed-source/commercial.**
Patent caveat: "Iteration batching is protected by our patents in the US and
Korea, and cannot be used without our authorization." Not enforced against
OSS but commercial users should be aware. Vendor benchmarks: "approximately
2-4× latency reduction" and "up to 6× higher throughput than vLLM."
URL: https://friendli.ai.

### Aphrodite Engine
**vLLM fork** maintained by PygmalionAI; **AGPL-3.0** (more restrictive than
vLLM's Apache-2.0; problematic for closed-source SaaS deploys). Distinguishing
strength: widest quantization matrix on the market — AQLM, AutoRound, AWQ,
BitNet, Bitsandbytes, EETQ, ExLlamaV3, GGUF, GPTQ, QuIP#, SqueezeLLM, Marlin,
FP2-FP12, NVIDIA ModelOpt, TorchAO, VPTQ, compressed_tensors, MXFP4, NVFP4,
plus standard FP8. Also ships creative-writing samplers (DRY, XTC, Mirostat).
Choose when: you need a quant format vLLM doesn't ship (BitNet, AQLM, QuIP#,
ExLlamaV3) and AGPL is acceptable.
Repo: https://github.com/aphrodite-engine/aphrodite-engine.

### llama.cpp / Ollama
**Not the A10G target.** llama.cpp is C/C++ inference with no dependencies;
reference implementation of GGUF; substrate of Ollama, LM Studio, Jan, etc.
Hardware: Apple Silicon (Metal), x86 (AVX/AMX), RISC-V, NVIDIA (CUDA), AMD
(HIP), Intel (SYCL), Vulkan. GGUF quantization: 1.5-bit through 8-bit
(Q2_K, Q4_K_M, Q5_K_M, Q6_K, Q8_0, IQ-series imatrix). `llama-server`:
"OpenAI API compatible HTTP server"; tool calling supported in Chat
Completions endpoint. Ollama: user-friendly llama.cpp wrapper at
ollama.com/library; tool-calling/JSON mode added 2024. Use case: edge,
desktop, CPU-only, or Apple Silicon — explicitly **out of scope** for this
wiki's A10G/g5.xlarge production target.
Repos: https://github.com/ggml-org/llama.cpp, https://github.com/ollama/ollama.

## Decision matrix (one-line summary)

| Need | Pick |
|---|---|
| Default open-source serving | [[infrastructure/vllm]] |
| Prefix-heavy / agentic / RAG | [[infrastructure/sglang]] |
| Largest model, smallest GPU (AWQ INT4) | [[infrastructure/lmdeploy]] |
| NVIDIA-peak, NVFP4/MXFP4 native | [[infrastructure/tensorrt-llm]] under [[infrastructure/nvidia-dynamo]] |
| Multi-replica with prefix-aware routing on K8s | [[infrastructure/vllm-production-stack]] |
| Full K8s operator (canary, multi-model, predictive+generative) | [[infrastructure/llm-d]] |
| Multi-host single model (TP×PP across nodes) | [[infrastructure/leaderworkerset]] |
| Multi-LoRA + heterogeneous GPUs + autoscaling | [[infrastructure/ray-serve-llm]] or [[infrastructure/aibrix]] |
| Disaggregated prefill/decode | [[infrastructure/nvidia-dynamo]] |
| CPU/Apple Silicon/edge | llama.cpp / Ollama (out of scope here) |

## Related

- [[infrastructure/vllm]]
- [[infrastructure/nvidia-dynamo]]
- [[concepts/parallelism-strategies]]
- [[concepts/disaggregated-serving]]
- [[concepts/serving-performance-measurement]]
- [[comparisons/serving-stacks-comparison]]
- [[comparisons/scaling-1-to-5-machines]]

## Sources

- [[sources/serving-stacks-alternatives-2026-05]]
