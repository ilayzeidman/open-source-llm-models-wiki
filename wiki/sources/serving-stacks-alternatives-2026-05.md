---
tags: [source, infrastructure, serving]
source_path: raw/serving-stacks-alternatives-2026-05.md
ingested: 2026-05-08
last_updated: 2026-05-08
---

# Alternative LLM serving stacks (May 2026)

A consolidated sweep of every credible open-source LLM serving stack that
competes with or complements vLLM + NVIDIA Dynamo. Each entry below has its
canonical URL, license, and key claims — used by the new infrastructure pages
under `wiki/infrastructure/`.

## Key claims

### Stack comparison (master row table)

| Stack | License | Primary value vs vLLM | Tool calling | Multi-node | A10G fit |
|---|---|---|---|---|---|
| **vLLM** | Apache-2.0 | Reference impl | Many model-aware parsers | Yes | Yes |
| **TensorRT-LLM** | Apache-2.0 | Peak NVIDIA perf; NVFP4/MXFP4 | Limited; via Dynamo | Yes | Yes |
| **SGLang** | Apache-2.0 | RadixAttention prefix reuse; CFSM JSON | 16+ parsers | Yes | Yes |
| **TGI** | Apache-2.0 | Maintenance mode (HF says migrate) | Generic Guidance | Limited | Yes |
| **LMDeploy** | Apache-2.0 | "1.8× vLLM" InternLM; AWQ-W4A16 specialty | InternLM2.5/Llama3.1 | Yes | Yes (excellent) |
| **NVIDIA Dynamo** | Apache-2.0 | Orchestrator over vLLM/SGLang/TRT-LLM | Inherits backend | Yes (designed) | Overkill single-GPU |
| **Dynamo-Triton** | BSD-3 | Multi-framework ML | n/a (per-backend) | Yes | Yes (general ML) |
| **Ray Serve LLM** | Apache-2.0 | Multi-model, multi-LoRA, autoscale over vLLM | Inherits vLLM | Yes | Yes |
| **DeepSpeed-FastGen** | Apache-2.0 | Dynamic SplitFuse | None documented | Limited | Yes (slowing) |
| **Friendli Engine** | Closed | Patent-holder iteration batching | Function-calling API | Yes | Commercial |
| **Aphrodite** | AGPL-3.0 | Widest quant matrix | Inherits vLLM | Limited | Yes |
| **llama.cpp / Ollama** | MIT | CPU/edge/Apple Silicon; GGUF | llama-server FC | No | Not target |
| **KServe** | Apache-2.0 | K8s + predictive ML CRDs | Per runtime | Yes | Yes |
| **vLLM Production Stack** | Apache-2.0 | Helm cluster + LMCache + router | Inherits vLLM | Yes | Yes |
| **llm-d** | Apache-2.0 | K8s Operator + KServe Inference Gateway | Inherits vLLM | Yes | Yes |

### Status flags

- **TGI** is in maintenance mode. HF README CAUTION: "text-generation-inference
  is now in maintenance mode... we contribute to and recommend using going
  forward: vllm, SGLang, as well as local engines with inter-compatibility
  such as llama.cpp or MLX."
- **NVIDIA Triton was renamed Dynamo-Triton** (March 2025) as part of the
  NVIDIA Dynamo Platform. Dynamo-Triton = formerly Triton (multi-framework);
  NVIDIA Dynamo = LLM-focused distributed inference.
- **DeepSpeed-FastGen**: latest release v0.3.3 (Mar 2025); pace of releases
  has slowed.
- **Friendli Engine**: closed-source. Holds patents on iteration batching;
  not enforced against OSS but commercial users should be aware.
- **Aphrodite**: AGPL-3.0 viral terms problematic for closed-source SaaS.
- **LLMPerf**: GitHub repo archived Dec 17 2025; methodology still cited.

### Distinguishing innovations

- **SGLang RadixAttention** — paper claim "up to 6.4× higher throughput" on
  agent control, logical reasoning, RAG, multi-turn. LRU radix tree across
  requests; automatic shared-prefix KV reuse.
- **DeepSpeed Dynamic SplitFuse** — long prompts decomposed across forward
  passes; short prompts fused to fill token budget. "up to 2.3× higher
  effective throughput than vLLM" (older bench).
- **FriendliAI Orca / iteration batching** — OSDI 2022 paper introduced the
  paradigm; "36.9× throughput improvement" vs FasterTransformer on GPT-3
  175B. Now industry standard, underpins every continuous-batching engine.
- **LMDeploy TurboMind AWQ-W4A16** — flagship, "delivers up to 1.8× higher
  request throughput than vLLM" on InternLM2-20B, "4-bit inference
  performance is 2.4× higher than FP16."
- **TensorRT-LLM "Architected on PyTorch"** — 1.0 release made the PyTorch
  path stable and default; tightly integrated with Dynamo as a backend.

### Tool-calling parser coverage (May 2026)

- **vLLM**: hermes, mistral, llama3_json, pythonic, llama4_pythonic, granite
  (3.x + 4), granite-20b-fc, internlm, jamba, xlam, deepseek_v3, deepseek_v31,
  qwen3_xml, openai (gpt-oss), kimi_k2, hunyuan_a13b, cohere_command3,
  longcat, glm45, glm47, functiongemma, olmo3, gigachat3, minimax.
- **SGLang**: deepseekv3, deepseekv31, deepseekv32, glm, glm45, glm47,
  gpt-oss, kimi_k2, llama3, llama4, mistral, pythonic, qwen, qwen25,
  qwen3_coder, step3. Plus separate `--reasoning-parser`.
- **TGI**: generic Guidance (grammar-based), no per-model parsers.
- **LMDeploy**: InternLM2.5, Llama 3.1 only.
- **Aphrodite**: inherits vLLM parsers in older forks; recent forks may have
  drifted.

### Key quotes

- vLLM positioning (Anyscale): "vLLM is only responsible for single model
  replicas, but for production deployments you often need an orchestration
  layer to autoscale, handle different fine-tuned adapters, handle distributed
  model-parallelism, and author multi-model, compound AI pipelines. Ray Serve
  is built to address the gaps that vLLM has for scaling and productionization."
- SGLang README: "high-performance serving framework for large language
  models and multimodal models... low-latency and high-throughput inference
  across a wide range of setups, from a single GPU to large distributed
  clusters."
- LMDeploy README: "delivers up to 1.8× higher request throughput than vLLM."
- HF TGI README: "text-generation-inference is now in maintenance mode."
- Dynamo developer page: "successor to NVIDIA Triton Inference Server"
  (specifically for generative AI).
- Dynamo-Triton: "deployment of AI models across major frameworks, including
  TensorRT, PyTorch, ONNX, OpenVINO, Python, and RAPIDS FIL."

## Pages updated on ingest

- [[infrastructure/serving-stack-landscape]] (NEW)
- [[infrastructure/sglang]] (NEW)
- [[infrastructure/tensorrt-llm]] (NEW)
- [[infrastructure/lmdeploy]] (NEW)
- [[infrastructure/tgi]] (NEW)
- [[infrastructure/triton-vs-dynamo]] (NEW)
- [[infrastructure/ray-serve-llm]] (NEW)
- [[infrastructure/vllm-production-stack]] (NEW)
- [[infrastructure/llm-d]] (NEW)
- [[infrastructure/aibrix]] (NEW)
- [[infrastructure/leaderworkerset]] (NEW)
- [[comparisons/serving-stacks-comparison]] (NEW)
- [[wiki/overview]]
- [[infrastructure/vllm]]
- [[infrastructure/nvidia-dynamo]]
