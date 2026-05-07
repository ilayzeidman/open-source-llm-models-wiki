---
tags: [source, models, microsoft, phi]
source_path: raw/phi-3-medium-128k-hf-card.md, raw/phi-4-hf-card.md
source_url: https://huggingface.co/microsoft/Phi-3-medium-128k-instruct, https://huggingface.co/microsoft/phi-4
ingested: 2026-05-07
last_updated: 2026-05-07
---

# Phi-3-Medium-128k + Phi-4 — Microsoft model cards

## Key claims

### Phi-3-Medium-128k-Instruct
- **HF model ID**: `microsoft/Phi-3-medium-128k-instruct`
- **Params**: 14B dense
- **Context**: 128K
- **License**: MIT (no commercial restriction)
- **Released**: 2024-05-21 (knowledge cutoff Oct 2023)

#### Benchmarks (model card)
- HumanEval 0-shot: 58.5
- MBPP 3-shot: 73.8
- MMLU 5-shot: 76.6
- GSM8K: 87.5
- BBH: 77.9
- BFCL: not on card

#### vLLM serving
- ⚠ **No native tool-call parser support.** Model card does not document a tool template; vLLM has no `phi3_*` parser. Tool use requires prompt-based hacks or fine-tunes — not first-class.
- Microsoft publishes ONNX variants (int4 DML, int4 CUDA, int4 CPU/Mobile, fp16 CUDA); many community AWQ/GGUF.

### Phi-4 (14B dense)
- **HF model ID**: `microsoft/phi-4`
- **Params**: 14B dense
- **Context**: **16K** (significantly less than Phi-3-medium-128k)
- **License**: MIT
- **Released**: 2024-12-12 (knowledge cutoff June 2024)

#### Benchmarks (Microsoft card)
- MMLU: 84.8 (vs Phi-3-medium 76.6)
- HumanEval: 82.6 (vs 58.5)
- MATH: 80.4 (vs 52.9)
- MGSM: 80.6
- GPQA: 56.1
- DROP: 75.5
- SimpleQA: 3.0 (notable factual-knowledge weakness flagged by Microsoft itself)
- BFCL v3: 40.8 (per pricepertoken mirror, 2026-05-07)

#### vLLM serving
- ⚠ **Not natively supported in base Phi-4 14B.** Phi-4-mini (3.8B) added native function calling, but it's a different smaller model. vLLM has no `phi4_*` tool parser as of mid-2026.
- Open issues: vLLM #14682 (Phi-4-mini), #15788.
- 149 community quantized variants (llama.cpp, LM Studio, Jan, Ollama).

### Successor / current state

- Phi-4 14B is Microsoft's current flagship 14B-class general model. Phi-4-reasoning and Phi-4-multimodal are newer specialized siblings.
- **For tool calling specifically, Phi-4-mini is the only Phi family member with first-class function-call support** — and at 3.8B it's a different deployment story.
- For agentic / tool-calling research on a single A10G, **neither Phi-3-medium nor Phi-4 14B is competitive with Llama-3.1-8B / Granite-3.3-8B / Mistral-Small-3.2** despite their strong reasoning numbers.

## Citation URLs
- [HF: Phi-3-medium-128k-instruct](https://huggingface.co/microsoft/Phi-3-medium-128k-instruct)
- [HF: microsoft/phi-4](https://huggingface.co/microsoft/phi-4)
- [microsoft/phi-4 discussion: function call support](https://huggingface.co/microsoft/phi-4/discussions/7)
- [Phi-4-mini function-calling guide](https://github.com/microsoft/PhiCookBook/blob/main/md/02.Application/07.FunctionCalling/Phi4/FunctionCallingBasic/README.md)

## Pages updated on ingest
- [[models/phi-3-medium-14b]]
- [[wiki/comparisons/tool-calling-models-on-a10g]]
