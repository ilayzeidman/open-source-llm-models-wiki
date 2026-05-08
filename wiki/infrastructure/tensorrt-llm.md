---
tags: [infrastructure, serving, tensorrt-llm, nvidia]
last_updated: 2026-05-08
source_count: 1
---

# NVIDIA TensorRT-LLM

NVIDIA's open-source library for optimizing LLM and visual-generation
inference on NVIDIA GPUs. Apache-2.0. Most relevant when (a) you want
NVIDIA-peak performance on H100/H200/B200/B300, (b) you need NVFP4 or MXFP4
native, or (c) you are deploying [[infrastructure/nvidia-dynamo]] with
TensorRT-LLM as the backend rather than [[infrastructure/vllm]].
[Source: [[sources/serving-stacks-alternatives-2026-05]]]

## What it is

> "Architected on PyTorch" with "high-level Python LLM API supporting
> deployments from a single GPU to multi-GPU or multi-node configurations."
> [README]

The 1.0 release (2025) made the PyTorch architecture stable and default. The
legacy C++ TensorRT-engine flow remains but is being de-emphasized. New
deploys should target the PyTorch path.

## Headline features

- **Custom kernels** for attention, GEMMs, and MoE.
- **Prefill-decode disaggregation** — first-class.
- **Wide expert parallelism** for large MoE.
- **Speculative decoding** ("Boost Llama 3.3 70B Inference Throughput 3×").
- **Prefix caching, CUDA graphs, LoRA** — standard set.
- **In-flight batching** — Orca-style continuous batching.

## Quantization (most extensive of any open engine)

- INT8, FP8, FP4 (NVFP4 + MXFP4 on Blackwell)
- AWQ INT4
- GPTQ
- SmoothQuant
- v1.1: "CuteDSL NVFP4 grouped GEMM integration for Blackwell"

## Disaggregated serving

- v1.1 (2025) added "Connector API for state transfer in disaggregated
  serving."
- v1.1 also added "KV cache reuse for MLA with host offloading."

These changes directly support [[infrastructure/nvidia-dynamo]]'s
disaggregated prefill/decode architecture.

## Tool calling

Not a first-class API surface in the README. Recent release notes mention:

- "fixes for harmony and tool-calling parsers… for agentic coding use cases"
- A Nemotron 3 Super deployment guide updated for tool calling and reasoning
  parsers.
- Added K2 (Kimi-K2) tool-calling examples.

In practice, **TensorRT-LLM is deployed under [[infrastructure/nvidia-dynamo]]**
or [[infrastructure/triton-vs-dynamo|Dynamo-Triton]], and Dynamo provides the
OpenAI-compatible chat-completions / tool-calling layer. If you want
plain-vLLM-style tool calling without an orchestrator, TensorRT-LLM is the
wrong choice.

## Performance claims (vendor)

- "Llama 4 at over 40,000 tokens per second on B200 GPUs."
- "Boost Llama 3.3 70B Inference Throughput 3×" (speculative decoding).
- No head-to-head vs vLLM in official docs. Real-hardware Dynamo+vLLM numbers
  on GB200 NVL72 (12,587 tok/s/GPU on Kimi K2.5 NVFP4 8k/1k) are the closest
  apples-to-apples comparison; TRT-LLM under Dynamo typically posts higher
  raw FLOPs/s but vLLM has caught up substantially with V1.

## v1.2 (Q4 2025)

- DGX Spark beta support.
- Validated MXFP4/FP8/NVFP4/FP16 across multiple architectures.
- NGC container release tracks `release` tag; tied to PyTorch 2.10/CUDA 13.1
  in current docs.

## When to choose TensorRT-LLM

- You're already on NVIDIA Dynamo and want the highest single-GPU throughput
  on H100/H200/B200.
- You need NVFP4 (Blackwell) or MXFP4 native at production speed.
- You have an existing TensorRT-engine-based pipeline you want to extend.

## When NOT to choose TensorRT-LLM

- You want a quick, OpenAI-compatible single-node serve — vLLM is faster
  to set up.
- You need specific tool-calling parser surface that vLLM/SGLang already
  expose; TRT-LLM's parser support is thinner without Dynamo on top.
- You're targeting Ampere (A10G/A100) — TensorRT-LLM works but vLLM gives
  similar performance with less ceremony, and the FP4/NVFP4/MXFP4 features
  that justify TRT-LLM aren't useful pre-Hopper.

## A10G fit

✅ — works on Ampere, but the marginal benefit over vLLM is small. The
TensorRT-LLM advantage compounds at H100/H200/B200; on A10G it's roughly a
wash with vLLM. Use vLLM unless you have a specific reason.

## Related

- [[infrastructure/nvidia-dynamo]] (TRT-LLM's most common deployment shape)
- [[infrastructure/triton-vs-dynamo]]
- [[infrastructure/serving-stack-landscape]]
- [[infrastructure/vllm]]

## Sources

- [[sources/serving-stacks-alternatives-2026-05]]
