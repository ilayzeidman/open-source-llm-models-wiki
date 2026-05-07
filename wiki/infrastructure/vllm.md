---
tags: [infrastructure, serving, vllm]
last_updated: 2026-05-07
source_count: 0
---

# vLLM

High-throughput LLM inference server, OpenAI-compatible API, paged-attention KV cache. The serving foundation under [[infrastructure/nvidia-dynamo]].

## What matters for this research

### Tool-call parsing

vLLM ships per-model parsers that translate the model's native tool-call output format into OpenAI-style `tool_calls` JSON. Enable with:

```
--enable-auto-tool-choice --tool-call-parser <name>
```

Parser families relevant here *(unverified — needs source)*:

| Parser | Models |
|---|---|
| `hermes` | Hermes-2/3 family ([[models/hermes-3-llama-3.1-8b]]) |
| `llama3_json` | Llama 3.1/3.2/3.3 instruct ([[models/llama-3.1-8b-instruct]], [[models/llama-3.3-70b-instruct]]) |
| `mistral` | Mistral-Instruct, Codestral, Mistral-Small ([[models/mistral-small-24b]], [[models/codestral-22b]]) |
| `granite` | IBM Granite ([[models/granite-3.1-8b]]) |
| `pythonic` | Some Llama-finetunes; emits Python-style calls |
| `internlm`, `jamba`, etc. | model-family specific |

Models without a dedicated parser still work via:
- A custom Jinja chat template + the `pythonic` or generic JSON parser.
- Constrained decoding via `xgrammar` / `outlines` / `lm-format-enforcer` to force valid JSON.

### Structured outputs / constrained decoding

Useful when tool-call format compliance is the bottleneck:
- **xgrammar** — vLLM's default since 2024; fast, GBNF-like grammar.
- **outlines** — Pythonic, slower historically.
- **lm-format-enforcer** — token-mask based.

Enable with `--guided-decoding-backend <xgrammar|outlines|...>`.

### Quantization backends

vLLM supports loading pre-quantized weights from Hugging Face. See [[infrastructure/quantization]].

| Backend | Format | A10G (Ampere) status |
|---|---|---|
| AWQ | INT4 weight-only | ✅ excellent (Marlin kernels) |
| GPTQ | INT4 weight-only | ✅ good (Marlin kernels) |
| INT8 SmoothQuant | W8A8 | ✅ ok |
| FP8 | W8A8 | ⚠ limited — Ampere lacks native FP8 tensor cores |
| BitsAndBytes | NF4/INT8 | ⚠ slower than AWQ/GPTQ for serving |
| GGUF | various | ❌ not vLLM-native (use llama.cpp instead) |

### Prefix caching

`--enable-prefix-caching` reuses KV cache across requests sharing a prompt prefix. Big win for tool-calling workloads where the system prompt + tool schemas are repeated every turn.

### Chunked prefill

`--enable-chunked-prefill` interleaves prefill and decode in a single batch, smoothing latency. On by default in recent vLLM versions.

### Speculative decoding

Smaller draft model proposes tokens, target model verifies. Useful for code generation where hit-rate is high. Vocab compatibility matters — Qwen draft + Qwen target works; cross-family draft pairs often don't.

## Memory tuning flags

| Flag | Purpose |
|---|---|
| `--gpu-memory-utilization 0.9` | fraction of VRAM vLLM may consume (default 0.9) |
| `--max-model-len <N>` | hard ceiling on context length; reducing this frees KV-cache budget |
| `--max-num-seqs <N>` | concurrent request cap |
| `--swap-space <GiB>` | CPU swap for KV cache (rarely useful at A10G scale) |
| `--enforce-eager` | disable CUDA graphs (debugging only; ~10–30% slower) |

## Tensor parallelism

`--tensor-parallel-size N` splits weights across N GPUs. On g5 (PCIe only) overhead is meaningful; on p4d/p5 (NVLink) it's nearly free. See [[hardware/multi-gpu-options]].

## Versioning notes

vLLM moves fast — tool parsers, quantization backends, and structured-output integrations get added or refactored in nearly every minor release. Always pin a specific version in production and re-test after upgrades. Treat any fact on this page as **needing re-verification against the version actually deployed**.

## Related
- [[infrastructure/nvidia-dynamo]]
- [[infrastructure/quantization]]
- [[concepts/tool-selection]]

## Sources
- (none yet)

## TODO / verify
- Latest vLLM release notes (current parser list)
- vLLM tool-calling docs page
- vLLM quantization support matrix
- xgrammar vs outlines benchmarks for tool-call formats
