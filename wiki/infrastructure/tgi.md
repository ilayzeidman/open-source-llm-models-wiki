---
tags: [infrastructure, serving, tgi, maintenance-mode]
last_updated: 2026-05-08
source_count: 1
---

# Hugging Face TGI (Text Generation Inference)

⚠ **Status (May 2026): MAINTENANCE MODE.** Hugging Face explicitly
recommends migration to [[infrastructure/vllm]], [[infrastructure/sglang]],
or llama.cpp/MLX for new deployments. Included in the wiki for completeness
and to explain why HF Inference Endpoints, Hugging Chat, and downstream
products still use it. [Source: [[sources/serving-stacks-alternatives-2026-05]]]

## Status (verbatim from README)

> "text-generation-inference is now in maintenance mode. Going forward, we
> will accept pull requests for minor bug fixes, documentation improvements
> and lightweight maintenance tasks."
>
> "TGI has initiated the movement for optimized inference engines to rely on
> a `transformers` model architectures. This approach is now adopted by
> downstream inference engines, which we contribute to and recommend using
> going forward: vllm, SGLang, as well as local engines with inter-
> compatibility such as llama.cpp or MLX."

This is HF's official position. **Treat new TGI deploys as legacy.**

## Latest release

v3.3.7 (Dec 19, 2025).

## What it still does

- "A Rust, Python and gRPC server for text generation inference."
- "Used in production at Hugging Face to power Hugging Chat, the Inference
  API and Inference Endpoints."
- Continuous batching of incoming requests.
- "Optimized transformers code for inference using Flash Attention and Paged
  Attention on the most popular architectures."
- Tensor parallelism.
- Token streaming via SSE.
- OpenAI-compatible Messages API.
- OpenTelemetry distributed tracing.
- Prometheus metrics.

## Quantization

bitsandbytes, GPT-Q, EETQ, AWQ, Marlin, fp8.

## Tool / function calling

Implemented via **"Guidance"** — grammar-based constrained decoding. Generic
across models. **Lacks the per-model parser awareness** of vLLM and SGLang —
no `qwen3_coder` / `kimi_k2` / `granite` / `glm45` / `mistral` parser. For
tool-calling-heavy workloads, this is the binding limitation.

## TGI v3.0 long-prompt claim

TGI v3.0 (Dec 2024) claimed: "**13× faster than vLLM on long prompts**." This
is real on the specific long-prompt benchmark TGI ran. The vLLM V1 engine
release (Jan 2025) and chunked prefill closed most of the gap; the comparison
is no longer current. Cite this number only with a date qualifier.

## Multi-backend support (Jan 2025)

TGI v3 added multi-backend dispatch: the same TGI front-end can route to
vLLM or TensorRT-LLM as backends. This is essentially a deprecation path —
TGI as a thin wrapper on top of the engines HF recommends migrating to.

## When to keep using TGI

- Existing production deployments on HF Inference Endpoints — already
  using it; no need to migrate without a reason.
- Hugging Chat or similar HF-hosted services.
- A TGI-specific feature you depend on (e.g., specific Guidance grammar
  feature, specific quant kernel).

## When to migrate off TGI

- New deployments — pick [[infrastructure/vllm]] or [[infrastructure/sglang]].
- Tool-calling-heavy workloads — TGI's generic Guidance is no match for
  vLLM's per-model parsers.
- Long-term roadmap dependability — TGI's pace will slow further.

## A10G fit

✅ — TGI runs on A10G. But the wiki's recommendation is to use vLLM (or SGLang)
on the same hardware.

## Related

- [[infrastructure/vllm]]
- [[infrastructure/sglang]]
- [[infrastructure/serving-stack-landscape]]

## Sources

- [[sources/serving-stacks-alternatives-2026-05]]
