# Alternative LLM serving stacks — May 2026 research sweep

Consolidated from official READMEs, docs, release notes, and engineering blogs.
Sources fetched 2026-05-08. Each entry below preserves canonical URLs and direct
quotes used as wiki citations.

---

## NVIDIA TensorRT-LLM

**Canonical URLs**
- GitHub: https://github.com/NVIDIA/TensorRT-LLM
- Docs: https://nvidia.github.io/TensorRT-LLM/
- Release notes: https://nvidia.github.io/TensorRT-LLM/release-notes.html
- Developer page: https://developer.nvidia.com/tensorrt-llm

**License**: Apache-2.0

**Key facts**
- "Architected on PyTorch" (1.0 release made the PyTorch path stable and default;
  legacy C++ TensorRT-engine flow remains but de-emphasized).
- "high-level Python LLM API" supporting deployments "from a single GPU to
  multi-GPU or multi-node configurations."
- Custom kernels for attention, GEMMs, and MoE.
- Features: "Prefill-Decode disaggregation, Wide Expert Parallelism,
  Speculative Decoding," prefix caching, CUDA graphs, LoRA.
- v1.1 (2025) added "Connector API for state transfer in disaggregated serving"
  and "KV cache reuse for MLA with host offloading."
- v1.2 (2025) added DGX Spark beta support; validated MXFP4/FP8/NVFP4/FP16.
- Quantization: INT8, FP8, FP4 (NVFP4 + MXFP4 on Blackwell), AWQ INT4, GPTQ,
  SmoothQuant.
- "CuteDSL NVFP4 grouped GEMM integration for Blackwell" (v1.1 notes).
- Tool calling: not first-class; recent notes mention "fixes for harmony and
  tool-calling parsers… for agentic coding use cases" and added K2 tool-calling
  examples. In practice TRT-LLM runs as a backend behind NVIDIA Dynamo, which
  provides the OpenAI-compatible chat-completions/tool-calling layer.
- Performance claim (no head-to-head vs vLLM in official docs): "Llama 4 at
  over 40,000 tokens per second on B200 GPUs" and "Boost Llama 3.3 70B
  Inference Throughput 3x" (speculative decoding).
- NGC container release tracks `release` tag; tied to PyTorch 2.10/CUDA 13.1
  in current docs.

---

## SGLang

**Canonical URLs**
- GitHub: https://github.com/sgl-project/sglang
- Docs: https://docs.sglang.io/
- Tool parser docs: https://docs.sglang.io/advanced_features/tool_parser.html
- RadixAttention paper (Zheng et al., 2023; revised 2024): https://arxiv.org/abs/2312.07104
- 2026 Q1 roadmap issue: https://github.com/sgl-project/sglang/issues/12780

**License**: Apache-2.0

**Key facts**
- "high-performance serving framework for large language models and multimodal
  models" — "low-latency and high-throughput inference across a wide range of
  setups, from a single GPU to large distributed clusters."
- Two distinguishing innovations:
  1. **RadixAttention** — LRU radix-tree of KV blocks across requests.
     Automatic prefix-cache reuse for shared system prompts, tool definitions,
     RAG contexts, multi-turn chats. Paper claim: "up to 6.4x higher throughput
     compared to existing inference systems" on agent control, logical reasoning,
     RAG, multi-turn.
  2. **Compressed finite state machine** for constrained decoding (JSON/regex/
     grammar) — README: "3x faster JSON decoding."
- Continuous batching, paged attention, chunked prefill, prefill-decode
  disaggregation, speculative decoding, "tensor/pipeline/expert/data parallelism,"
  zero-overhead CPU scheduler.
- Quantization: "FP4/FP8/INT4/AWQ/GPTQ" (README).
- Tool calling **first-class**. Supported `--tool-call-parser` values
  (May 2026): `deepseekv3`, `deepseekv31`, `deepseekv32`, `glm`, `glm45`, `glm47`,
  `gpt-oss`, `kimi_k2`, `llama3`, `llama4`, `mistral`, `pythonic`, `qwen`,
  `qwen25`, `qwen3_coder`, `step3`. Separate `--reasoning-parser` for thinking-
  mode models (`deepseek-v3`, `qwen3`, `kimi_k2`).
- Day-0 model support stated: "MiMo-V2-Flash, Nemotron 3 Nano, Mistral Large 3,
  LLaDA 2.0 Diffusion LLM, MiniMax M2."
- Adopters listed in README: "xAI, AMD, NVIDIA, Intel, LinkedIn, Cursor,
  Oracle Cloud, Google Cloud, Microsoft Azure, AWS."
- Third-party benchmarks (particula.tech, runpod.io, spheron.network):
  ~29% throughput edge on H100 in steady state; "up to 6.4x" on prefix-heavy
  workloads. Smaller throughput degradation than vLLM under guided decoding
  (vLLM "shows noticeable degradation at batch sizes of 8 and above").

---

## Hugging Face Text Generation Inference (TGI)

**Canonical URLs**
- GitHub: https://github.com/huggingface/text-generation-inference
- Docs: https://huggingface.co/docs/text-generation-inference/en/index
- Latest release: v3.3.7 (Dec 19, 2025)

**License**: Apache-2.0

**Key facts**
- "A Rust, Python and gRPC server for text generation inference. Used in
  production at Hugging Face to power Hugging Chat, the Inference API and
  Inference Endpoints."
- **Status (May 2026): MAINTENANCE MODE.** README CAUTION block:
  > "text-generation-inference is now in maintenance mode. Going forward, we
  > will accept pull requests for minor bug fixes, documentation improvements
  > and lightweight maintenance tasks."
  > "TGI has initiated the movement for optimized inference engines to rely on
  > a `transformers` model architectures. This approach is now adopted by
  > downstream inference engines, which we contribute to and recommend using
  > going forward: vllm, SGLang, as well as local engines with
  > inter-compatibility such as llama.cpp or MLX."
- Hugging Face explicitly recommends migration off TGI to vLLM, SGLang,
  llama.cpp, or MLX. New TGI deploys (2026) should be considered legacy.
- Features (still functional): continuous batching; "Flash Attention and
  Paged Attention" optimized kernels; tensor parallelism; SSE token streaming;
  OpenAI-compatible Messages API; OpenTelemetry tracing; Prometheus metrics.
- Quantization: bitsandbytes, GPT-Q, EETQ, AWQ, Marlin, fp8.
- Tool/function calling supported via "Guidance" feature (grammar-based,
  generic; lacks per-model parser awareness of vLLM/SGLang).
- TGI v3.0 release (Dec 2024) claimed "13x faster than vLLM on long prompts" —
  long-prompt-specific; vLLM V1 + chunked-prefill closed most of the gap.

---

## LMDeploy

**Canonical URLs**
- GitHub: https://github.com/InternLM/lmdeploy
- Docs: https://lmdeploy.readthedocs.io/
- TurboMind config: https://github.com/InternLM/lmdeploy/blob/main/docs/en/inference/turbomind_config.md

**License**: Apache-2.0

**Key facts**
- "toolkit for compressing, deploying, and serving LLM" by MMRazor + MMDeploy
  (InternLM ecosystem).
- Two engines:
  - **TurboMind** (C++/CUDA) — flagship; "ultimate optimization of inference
    performance."
  - **PyTorch** — pure Python; lower barrier.
- Features: "persistent batch (a.k.a. continuous batching), blocked KV cache,
  dynamic split & fuse, tensor parallelism, high-performance CUDA kernels."
- Headline performance claim: "delivers up to 1.8x higher request throughput
  than vLLM" (on InternLM2-20B). 4-bit AWQ: "4-bit inference performance is
  2.4x higher than FP16."
- Quantization: AWQ W4A16 is the flagship; supports running KV cache
  quantization + AWQ + automatic prefix caching **simultaneously** (most
  engines forbid combinations).
- 2025/09 release: "TurboMind MXFP4 on NVIDIA GPUs starting from V100, achieving
  1.5x the performance of vLLM on H800 for openai gpt-oss models."
- Recent updates: 2025/01 DeepSeek V3 + R1; 2025/04 FlashMLA + DeepGemm;
  2026/02 Qwen3.5 + llm-compressor 4-bit.
- Tool calling: function calling for InternLM2.5 and Llama 3.1 (8B/70B).
  Less polished than SGLang/vLLM in parser breadth.
- Multi-GPU/multi-node: "easy and efficient deployment of multi-model services
  across multiple machines and cards." TP works; MoE EP added recently.
- **Wiki sweet spot**: AWQ-W4A16 on a single A10G. Built around fitting big
  models on small GPUs via aggressive 4-bit quantization.

---

## NVIDIA Triton Inference Server vs NVIDIA Dynamo

**Canonical URLs**
- Dynamo: https://developer.nvidia.com/dynamo
- Dynamo-Triton: https://developer.nvidia.com/dynamo-triton
- Newsroom (GTC 2025 Dynamo announcement):
  https://nvidianews.nvidia.com/news/nvidia-dynamo-open-source-library-accelerates-and-scales-ai-reasoning-models
- Combined product page: https://www.nvidia.com/en-us/ai/dynamo-triton/

**Naming change (March 2025)**
- NVIDIA Triton Inference Server was **renamed** to **NVIDIA Dynamo-Triton** as
  part of the broader NVIDIA Dynamo Platform.
- The brand "Dynamo" now spans two distinct products:

### Dynamo-Triton (formerly Triton Inference Server)
- General-purpose, multi-framework serving platform.
- "enables deployment of AI models across major frameworks, including
  TensorRT, PyTorch, ONNX, OpenVINO, Python, and RAPIDS FIL. It delivers high
  performance with dynamic batching, concurrent execution, and optimized
  configurations."
- Targets traditional ML, vision, recommendation, ensemble, and audio/video
  streaming workloads.
- Runs on NVIDIA GPUs, non-NVIDIA accelerators, x86, and ARM CPUs.

### NVIDIA Dynamo
- LLM-focused, distributed inference framework.
- "an open source, low-latency, modular inference framework for serving
  generative AI models in distributed environments."
- "successor to NVIDIA Triton Inference Server" specifically for generative
  AI / reasoning workloads.
- Open-source on GitHub; production support via NVIDIA AI Enterprise.

### Dynamo's four key components
- **SLO Planner** — "monitors capacity and prefill activity in multi-node
  deployments adjusting GPU resources to consistently meet Service Level
  Objectives (SLO)."
- **KV-aware Router** — "efficiently directs incoming traffic across large
  GPU fleets in multi-node deployments to minimize redundant KV Cache
  re-computations."
- **NIXL** — "low latency point-to-point inference data transfer library that
  accelerates the transfer of KV cache between GPUs and across heterogeneous
  memory and storage types."
- **KV Block Manager** — "cost-aware KV caching engine that transfers KV cache
  across various memory hierarchies, freeing up GPU memory while maintaining
  user experience."

### Backends (Dynamo)
"open source inference engines including SGLang, TensorRT-LLM, and vLLM."
Dynamo is not itself an inference engine — it orchestrates them.

### Performance claims
- "Using the same number of GPUs, Dynamo doubles the performance and revenue
  of AI factories serving Llama models on today's NVIDIA Hopper platform."
- "running the DeepSeek-R1 model on a large cluster of GB200 NVL72 racks,
  NVIDIA Dynamo's intelligent inference optimizations also boost the number
  of tokens generated by over 30x per GPU."

### Decision rule
- LLM/agentic/reasoning, multi-node, KV-cache-heavy → **Dynamo**.
- Mixed-framework (XGBoost + PyTorch + ensemble + audio/video) or single-node
  TensorRT engine serving → **Dynamo-Triton**.
- Single-node single-model vLLM, no orchestration → just **vLLM**.

---

## Ray Serve LLM (Anyscale)

**Canonical URLs**
- Anyscale announcement: https://www.anyscale.com/blog/llm-apis-ray-data-serve
- Wide-EP + disaggregated serving (Nov 2025):
  https://www.anyscale.com/blog/ray-serve-llm-anyscale-apis-wide-ep-disaggregated-serving-vllm
- Migration guide: https://docs.anyscale.com/llms/serving/migration_guide_to_ray_serve_llm/
- Ray docs: https://docs.ray.io/en/latest/data/working-with-llms.html
- Archived RayLLM: https://github.com/ray-project/ray-llm

**License**: Apache-2.0

**Key facts**
- Production-grade LLM-serving API built on Ray Serve, replacing legacy
  `RayLLM` (now archived).
- Introduced in Ray ≥ 2.44.0; "deep integration with inference engines (vLLM
  to start)."
- Anyscale positioning: "vLLM is only responsible for single model replicas,
  but for production deployments you often need an orchestration layer to
  autoscale, handle different fine-tuned adapters, handle distributed
  model-parallelism, and author multi-model, compound AI pipelines."
- Features:
  - "Automatic scaling and load balancing"
  - "Unified multi-node multi-model deployment"
  - "OpenAI compatibility"
  - "Multi-LoRA support with shared base models"
  - Pythonic API: `build_openai_app`, `LLMConfig`, `LLMServer`.
- November 2025 additions:
  - `build_dp_deployment` for **wide expert parallelism** (MoE experts spread
    across many GPUs with replicated attention).
  - `build_pd_openai_app` for **prefill/decode disaggregation** — `PDProxyServer`
    issues prompt prefill with `max_tokens=1` to populate KV cache, then hands
    off to decode deployment.
- Use case fit: multi-model deployments, multi-LoRA (shared base + many
  adapters), autoscaling on heterogeneous GPU clusters, single Python definition
  for compound AI pipelines.

---

## DeepSpeed-MII / DeepSpeed-FastGen

**Canonical URLs**
- Blog: https://github.com/microsoft/DeepSpeed/blob/master/blogs/deepspeed-fastgen/README.md
- Repo: https://github.com/deepspeedai/DeepSpeed-MII (mirror: microsoft/DeepSpeed-MII)
- Paper: https://arxiv.org/abs/2401.08671 ("DeepSpeed-FastGen: High-throughput
  Text Generation for LLMs via MII and DeepSpeed-Inference")

**License**: Apache-2.0

**Key facts**
- LLM-serving system from **DeepSpeed-MII** (Python + RESTful gateway) +
  **DeepSpeed-Inference** (CUDA kernels).
- Distinguishing innovation: **Dynamic SplitFuse**.
  > "Long prompts are decomposed into smaller chunks across multiple forward
  > passes" and "Short prompts are fused to precisely fill a target token
  > budget."
- Performance claims vs vLLM (official blog):
  - "up to 2.3x higher [effective] throughput than vLLM" (1.42 vs 0.63 q/s)
  - "up to 2x higher throughput (1.36 rps vs. 0.67 rps)" at identical latency
  - "up to 50% latency reduction (7 seconds vs. 14 seconds)"
  - P95 generation-latency tail "3.7 times" smaller
  - Near-linear 16x throughput scaling with 16 replicas
- Supported model families: Llama, Llama-2, Llama-3, Mistral, Mixtral, OPT,
  Falcon, Phi-2, Phi-3, Qwen, Qwen2, Qwen2-MoE.
- Multi-replica / RESTful gateway: built-in HTTP server with multi-replica
  load balancing. TP + replicas combinable.
- **Status (May 2026)**: latest release v0.3.3 dated March 25, 2025. Pace of
  releases slowed; functional but no longer cutting-edge for newest models.
  Microsoft's recent LLM serving investment is in DeepSpeed-Inference kernels,
  not MII.
- Tool calling: not a documented feature.

---

## Friendli Engine (FriendliAI)

**Canonical URLs**
- Product blog: https://friendli.ai/blog/friendli-serve-transformer-models
- vLLM comparison: https://friendli.ai/blog/comparing-friendli-engine-vllm
- Iteration batching post: https://friendli.ai/blog/llm-iteration-batching
- Orca research page: https://friendli.ai/research/orca
- Orca paper (OSDI 2022): https://www.usenix.org/conference/osdi22/presentation/yu

**License**: **Closed-source / commercial.**

**Key facts**
- Conceptual seed for almost every continuous-batching system in OSS.
- "Friendli Inference was born out of our Orca research; the Orca paper was
  published in OSDI 2022."
- Orca (Yu et al., FriendliAI + Seoul National University, OSDI 2022)
  introduced:
  - **Iteration-level scheduling** ("a new scheduling mechanism that schedules
    execution at the granularity of iteration… instead of request").
  - **Selective batching** (apply batching only to selected ops to mix
    sequences of different lengths).
- Reported result: **"36.9x throughput improvement at the same level of
  latency"** vs NVIDIA FasterTransformer on GPT-3 175B.
- Ideas now industry-standard; underpin vLLM, TGI, TensorRT-LLM, SGLang,
  DeepSpeed-FastGen, LMDeploy, Aphrodite. FriendliAI states: "ORCA inspired
  vLLM and several other open-source frameworks."
- **Patent caveat**: "Iteration batching is protected by our patents in the
  US and Korea, and cannot be used without our authorization." Patent has not
  been enforced against OSS, but commercial users should be aware.
- Engine vendor benchmarks (not peer-reviewed): "approximately 2x to 4x
  latency reduction" and "up to 6x higher throughput than vLLM."
- Form factors: Friendli Container (self-hosted), Dedicated Endpoints,
  Serverless Endpoints.
- Tool calling: OpenAI-compatible API supports function-calling at protocol
  level; not described in detail in docs we sampled.

---

## Aphrodite Engine

**Canonical URLs**
- GitHub: https://github.com/aphrodite-engine/aphrodite-engine
- Docs site: https://aphrodite.pygmalion.chat/
- Quantization wiki: https://github.com/aphrodite-engine/aphrodite-engine/wiki/8.-Quantization

**License**: **AGPL-3.0** (more restrictive than vLLM's Apache-2.0; matters
for commercial deploys).

**Key facts**
- "an inference engine that optimizes the serving of HuggingFace-compatible
  models at scale" — vLLM fork: "builds upon and integrates the exceptional
  work from various projects, primarily vLLM."
- Distinguishing strength: **quantization breadth**. Supports AQLM, AutoRound,
  AWQ, BitNet, Bitsandbytes, EETQ, ExLlamaV3, GGUF, GPTQ, QuIP#, SqueezeLLM,
  Marlin, FP2-FP12, NVIDIA ModelOpt, TorchAO, VPTQ, compressed_tensors, MXFP4,
  NVFP4, plus standard FP8.
- "NVFP4 — the new datatype supported by Blackwell GPUs, also working on
  Ampere and Hopper using Marlin kernels."
- BitBLAS-backed BitNet 1.58-bit.
- Other features: continuous batching with paged-attention KV cache; optimized
  CUDA kernels; distributed and disaggregated inference; speculative decoding
  (EAGLE, DFlash, ngram, MTP); multi-LoRA; multimodal; modern samplers (DRY,
  XTC, Mirostat — for roleplay).
- Multi-GPU: TP + PP (inherited from vLLM).
- Tool calling: not a documented headline feature; would inherit vLLM's parser
  surface in older forks; README doesn't enumerate.
- **When to choose**: need a quant format vLLM doesn't ship (AQLM, ExLlamaV3,
  QuIP#, BitNet); need creative-writing samplers; AGPL acceptable.

---

## llama.cpp / Ollama

**Canonical URLs**
- llama.cpp: https://github.com/ggml-org/llama.cpp (repo moved from
  `ggerganov/llama.cpp` to `ggml-org/llama.cpp`)
- Quantize tool: https://github.com/ggml-org/llama.cpp/blob/master/tools/quantize/README.md
- Ollama: https://github.com/ollama/ollama (https://ollama.com)

**License**: MIT (both)

**Key facts**
- llama.cpp: "LLM inference in C/C++" with "Plain C/C++ implementation without
  any dependencies." Reference impl of GGUF format; substrate of nearly every
  consumer-grade local LLM tool.
- Hardware support is unmatched:
  - Apple Silicon: ARM NEON, Accelerate, Metal.
  - x86: AVX, AVX2, AVX512, AMX.
  - RISC-V: RVV, ZVFH, ZFH, ZICBOP, ZIHINTPAUSE.
  - GPUs: NVIDIA (CUDA), AMD (HIP), Moore Threads (MUSA), Vulkan, SYCL.
- Quantization (GGUF): "1.5-bit, 2-bit, 3-bit, 4-bit, 5-bit, 6-bit, and 8-bit
  integer quantization" — Q2_K, Q3_K_M, Q4_K_M, Q5_K_M, Q6_K, Q8_0,
  IQ-series imatrix, ternary BitNet 1.58-bit experimentally. Q4_K_M default.
- HTTP server: `llama-server` — "A lightweight, OpenAI API compatible, HTTP
  server for serving LLMs" with "multiple concurrent users, parallel decoding,
  speculative decoding, embedding models, and grammar constraints." Tool
  calling supported in Chat Completions endpoint (model-template-driven).
- Ollama: user-friendly llama.cpp wrapper. `ollama run <model>` CLI; HTTP API;
  curated library at ollama.com/library. Backend: llama.cpp. Tool-calling/
  JSON-mode added 2024.
- **Wiki fit**: NOT a candidate for A10G/g5.xlarge production target. Edge/
  CPU/desktop tools.

---

## KServe

**Canonical URL**: https://kserve.github.io/website/

**License**: Apache-2.0 (CNCF; Linux Foundation)

**Key facts**
- "Standardized Distributed Generative and Predictive AI Inference Platform
  for Scalable, Multi-Framework Deployment on Kubernetes."
- K8s CRD project (sklearn/XGBoost/PyTorch/TF/ONNX/HF transformers).
- Features: scale-to-zero, autoscaling, canary rollouts, A/B testing,
  request-based autoscaling, KV-cache offloading to CPU/disk.
- Unified API for predictive + generative inference.
- Supported LLM runtimes: "vLLM and llm-d for high-performance LLM serving"
  + "Native HuggingFace model support."
- New `LLMInferenceService` CRD is the LLM-specific surface, introduced with
  the llm-d integration.

---

## vLLM Production Stack

**Canonical URLs**
- GitHub: https://github.com/vllm-project/production-stack
- Docs: https://docs.vllm.ai/en/latest/deployment/integrations/production-stack/
- KV-aware tutorial: https://docs.vllm.ai/projects/production-stack/en/vllm-stack-0.1.5/tutorials/kvaware.html
- LMCache blog (production-stack launch, 2025-01-21):
  https://blog.lmcache.ai/en/2025/01/21/high-performance-and-easy-deployment-of-vllm-in-k8s-with-vllm-production-stack/
- Router release blog (2025-12-13):
  https://blog.vllm.ai/2025/12/13/vllm-router-release.html

**License**: Apache-2.0

**Key facts**
- Released January 2025 by the vLLM team.
- "vLLM Production Stack represents the next step in transforming vLLM from a
  best-in-class single-node engine into a full-scale LLM serving system… an
  open-source reference implementation of an inference stack built on top of
  vLLM, designed to run seamlessly on a cluster of GPU nodes."
- Components:
  - **Request Router** — directs requests by routing key / session ID for
    KV-cache reuse; K8s service-discovery.
  - **LMCache integration** — KV cache offloading to CPU/disk; prefix-aware
    routing.
  - **Observability** — Prometheus + Grafana with TTFT distribution, KV cache
    hit rate, running/pending request counts, per-instance QPS.
  - **Helm-deployable**: `helm install vllm vllm/vllm-stack -f values-…yaml`.
  - OpenAI-compatible at the cluster boundary.
- Routing algorithms: round robin; session-ID stickiness; **prefix-aware load
  balancing**; KV-cache aware.
- Performance claim: "10x better performance with 3-10x lower response delay
  & 2-5x higher throughput" (vendor).
- Router v0.5 (Dec 2025) additions: hierarchical KV offloading; cache-aware
  LoRA routing; active-active HA; scale-to-zero autoscaling; UCCL transport
  resilience. Validated: ~3.1k tok/s/B200 decode (wide-EP); 50k output tok/s
  on 16×16 B200 P/D topology; "order-of-magnitude TTFT reduction vs
  round-robin baseline."

---

## llm-d

**Canonical URLs**
- Site: https://llm-d.ai/
- GitHub: https://github.com/llm-d/llm-d
- Architecture: https://llm-d.ai/docs/architecture
- Production blog: https://llm-d.ai/blog/production-grade-llm-inference-at-scale-kserve-llm-d-vllm

**License**: Apache-2.0

**Key facts**
- Kubernetes Operator built jointly with the KServe community for production
  LLM serving on K8s.
- Releases: 0.4 (2025), 0.5 (2026), 0.7 current (Apr 2026).
- Exposes `LLMInferenceService` and `LLMInferenceConfig` CRDs.
- Integrates with KServe's "Inference Gateway Extension" (Envoy + Gateway API
  Inference Extension) for "prefix-cache aware routing."
- Endpoint Picker (EPP) + InferencePool CRD — "scores and selects model server
  pods based on real-time metrics, KV-cache affinity, and configured policies."
- Reported customer gain: "3x improvement in output tokens/s" and "2x reduction
  in time to first token."

---

## Cross-cutting comparison table

| Stack | License | Primary value vs vLLM | Tool-calling parsers | Quantization | Multi-GPU | Multi-node | A10G fit |
|---|---|---|---|---|---|---|---|
| **vLLM** | Apache-2.0 | Reference impl | Many (model-aware) | FP8/AWQ/GPTQ/INT4/FP4 | Yes (TP/PP/DP/EP) | Yes | Yes |
| **TensorRT-LLM** | Apache-2.0 | NVIDIA peak perf, NVFP4/MXFP4 | Limited; via Dynamo | FP8/FP4/NVFP4/MXFP4/AWQ/GPTQ/SmoothQuant | Yes | Yes | Yes |
| **SGLang** | Apache-2.0 | RadixAttention prefix reuse; structured outputs | 16+ (deepseek*, glm*, kimi_k2, llama3/4, mistral, qwen*, gpt-oss, step3, pythonic) | FP4/FP8/INT4/AWQ/GPTQ | Yes (TP/PP/EP/DP) | Yes | Yes |
| **TGI** | Apache-2.0 | (Maintenance mode — HF recommends migration) | Generic via Guidance | bnb/GPT-Q/EETQ/AWQ/Marlin/fp8 | Yes (TP) | Limited | Yes |
| **LMDeploy** | Apache-2.0 | "1.8x vLLM" InternLM; AWQ-W4A16 specialty | InternLM2.5, Llama3.1 | AWQ-W4A16 (flagship), GPTQ, FP8, MXFP4, KV-quant | Yes | Yes | Yes (excellent) |
| **NVIDIA Dynamo** | Apache-2.0 | Orchestrator over vLLM/SGLang/TRT-LLM; PD disagg | Inherits backend's | Inherits | Yes | Yes (designed) | Possible but overkill |
| **Dynamo-Triton** | BSD-3 | Multi-framework general inference | n/a | per-backend | Yes | Yes | Yes (general ML) |
| **Ray Serve LLM** | Apache-2.0 | Orchestration over vLLM; multi-model, multi-LoRA, autoscale | Inherits vLLM | Inherits vLLM | Yes | Yes | Yes |
| **DeepSpeed-FastGen** | Apache-2.0 | Dynamic SplitFuse; "2.3x vLLM" (older bench) | None documented | FP16/INT8 | Yes (TP) | Limited | Yes (slowing) |
| **Friendli Engine** | Closed | Patent-holder iteration batching; "2-4x vLLM" (vendor) | Function-calling API | FP8/INT8/AWQ | Yes | Yes | Yes (commercial) |
| **Aphrodite** | AGPL-3.0 | Widest quant matrix; roleplay samplers | Inherits vLLM | AQLM, AutoRound, AWQ, BitNet, GGUF, GPTQ, QuIP#, ExLlamaV3, MXFP4, NVFP4, FP2-FP12, … | Yes | Limited | Yes |
| **llama.cpp / Ollama** | MIT | CPU/edge/Apple Silicon; GGUF | llama-server function-call | GGUF Q2-Q8, IQ-series, ternary | Limited | No | Not target |
| **KServe** | Apache-2.0 | K8s LLM + predictive serving | via runtime | per-runtime | Yes | Yes | Yes |
| **vLLM Production Stack** | Apache-2.0 | K8s helm vLLM cluster + LMCache + router | Inherits vLLM | Inherits | Yes | Yes | Yes |
| **llm-d** | Apache-2.0 | K8s Operator for vLLM + KServe Inference Gateway | Inherits vLLM | Inherits | Yes | Yes | Yes |
