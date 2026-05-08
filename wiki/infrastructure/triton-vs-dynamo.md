---
tags: [infrastructure, nvidia, dynamo, triton, naming]
last_updated: 2026-05-08
source_count: 2
---

# Triton vs Dynamo — naming disambiguation

NVIDIA's brand split is easy to confuse. As of March 2025 the original
**Triton Inference Server** was renamed **Dynamo-Triton**, and a separate,
new product called **NVIDIA Dynamo** was launched specifically for LLMs and
distributed inference. The "Dynamo" brand now spans two products. This page
disambiguates them. [Source: [[sources/serving-stacks-alternatives-2026-05]]]

## At a glance

| | NVIDIA Dynamo | Dynamo-Triton (formerly Triton Inference Server) |
|---|---|---|
| **Renamed from** | (new product, GTC 2025) | NVIDIA Triton Inference Server |
| **Primary domain** | LLMs / generative AI / reasoning | All ML — vision, recommender, ensemble, audio/video |
| **Architecture** | Orchestrator above engines | Multi-framework serving runtime |
| **Backend engines** | vLLM, SGLang, TensorRT-LLM | TensorRT, PyTorch, ONNX, OpenVINO, Python, RAPIDS FIL |
| **Distinguishing feature** | Disaggregated prefill/decode, KV-aware routing, NIXL, KVBM | Dynamic batching, concurrent execution, multi-model ensemble |
| **K8s native** | Yes (DGDR CRD, Grove gang scheduler) | Yes |
| **License** | Apache-2.0 (open source) | BSD-3 |
| **OpenAI-compatible API** | First-class | Via custom Python backend |
| **Hardware** | NVIDIA GPUs | NVIDIA GPUs, non-NVIDIA accelerators, x86, ARM CPUs |
| **Wiki page** | [[infrastructure/nvidia-dynamo]] | this page (no separate dedicated page) |

## Decision rule

| Workload | Pick |
|---|---|
| LLM / agentic / reasoning, multi-node, KV-cache heavy | **NVIDIA Dynamo** |
| Mixed framework (XGBoost + PyTorch + ensemble + audio/video pipelines) | **Dynamo-Triton** |
| Single-node TensorRT engine serving | **Dynamo-Triton** |
| Single-node, single-model vLLM, no orchestration | just **vLLM** (no Triton/Dynamo needed) |
| Replicate vLLM across K8s nodes with KV-aware routing | [[infrastructure/vllm-production-stack]] (lighter than Dynamo) |

## Quotes

### NVIDIA Dynamo (developer.nvidia.com/dynamo)
> "an open source, low-latency, modular inference framework for serving
> generative AI models in distributed environments... successor to NVIDIA
> Triton Inference Server."

### Dynamo-Triton (developer.nvidia.com/dynamo-triton)
> "enables deployment of AI models across major frameworks, including
> TensorRT, PyTorch, ONNX, OpenVINO, Python, and RAPIDS FIL. It delivers
> high performance with dynamic batching, concurrent execution, and
> optimized configurations."

## Why this matters for the wiki

The original wiki was written before the rename. References to "Triton" in
older sources may mean either product depending on date and context:

- Pre-2025 "Triton": always means Triton Inference Server (now Dynamo-Triton).
- 2025+ "Triton": ambiguous; check context. NVIDIA blog posts post-GTC 2025
  consistently use "Dynamo-Triton" or specify.
- Pre-2025 "Dynamo": almost certainly **not** the inference framework
  (project announced GTC 2025).

Older NVIDIA blog posts that cite "Triton + TRT-LLM for LLMs" are
**superseded** in 2026. New deployments should use **NVIDIA Dynamo**
specifically (with vLLM/SGLang/TRT-LLM as backend), not Dynamo-Triton.

## Related

- [[infrastructure/nvidia-dynamo]] — the LLM-focused product
- [[infrastructure/tensorrt-llm]] — the most common Dynamo backend
- [[infrastructure/vllm]]
- [[infrastructure/serving-stack-landscape]]

## Sources

- [[sources/serving-stacks-alternatives-2026-05]]
- [[sources/nvidia-dynamo-multinode-2026-05]]
