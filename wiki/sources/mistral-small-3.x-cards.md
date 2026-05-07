---
tags: [source, models, mistral]
source_path: raw/mistral-small-24b-2501-hf-card.md, raw/mistral-small-3-announcement.md, raw/mistral-small-3.1-hf-card.md, raw/mistral-small-3.1-announcement.md, raw/mistral-small-3.2-hf-card.md
source_url: https://huggingface.co/mistralai/Mistral-Small-3.2-24B-Instruct-2506
ingested: 2026-05-07
last_updated: 2026-05-07
---

# Mistral Small 3.x family — model cards & announcements

The Apache-2.0 24B-class generalist with strong tool calling. Three releases in 13 months. **Latest: 3.2 (2506).**

## Key claims

### Versions

| Slug | HF ID | Released | Context |
|---|---|---|---|
| 3.0 (2501) | `mistralai/Mistral-Small-24B-Instruct-2501` | 2025-01-30 | 32K |
| 3.1 (2503) | `mistralai/Mistral-Small-3.1-24B-Instruct-2503` | 2025-03-17 | **128K** |
| 3.2 (2506) | `mistralai/Mistral-Small-3.2-24B-Instruct-2506` | 2025-06 | **128K** |

All 24B dense, all **Apache-2.0** (commercial unrestricted), all use the Tekken tokenizer (131k vocab).

### Benchmarks

| Benchmark | 3.0 (2501) | 3.1 (2503) | 3.2 (2506) |
|---|---|---|---|
| HumanEval | 84.8 | 88.4 | — (see HE+) |
| HumanEval Plus pass@5 | — | 88.99 | 92.90 |
| MBPP | — | 74.7 | — |
| MBPP Plus pass@5 | — | 74.63 | 78.33 |
| MMLU | — | 80.62 | 80.50 |
| MMLU-Pro 5-shot CoT | — | 66.76 | 69.06 |
| MATH | — | 69.30 | 69.42 |
| GPQA Diamond | — | 45.96 | 46.13 |
| Wildbench v2 | — | 55.6 | 65.33 |
| Arena Hard v2 | — | 19.56 | 43.10 |
| IFEval (internal) | — | 82.75 | 84.78 |

**No BFCL number published in any official card.** Mistral's "agentic" claim is qualitative ("excellent at function/tool calling tasks via vLLM").

### vLLM serving (3.2 is the recommended baseline)

```
vllm serve mistralai/Mistral-Small-3.2-24B-Instruct-2506 \
  --tokenizer_mode mistral --config_format mistral --load_format mistral \
  --tool-call-parser mistral --enable-auto-tool-choice
```

The 3.2 card highlights the headline change vs 3.1: "**more robust function calling template**" for the `mistral` tool parser. Recommended temperature 0.15. Card warns: "Transformers implementation is untested; vLLM recommended for production use."

### Quant checkpoints

Card recommends BF16/FP16 (~55 GB) — but for A10G (24 GB) you need 4-bit:
- Community quants: `bartowski/...-GGUF`, `lmstudio-community/...-GGUF`, `unsloth/...-bnb-4bit`, `RedHatAI/Mistral-Small-3.2-24B-Instruct-2506-NVFP4`
- AWQ/GGUF 4-bit: ~13–14 GB weights — fits comfortably with KV-cache headroom for 30k+ context.

### Successor status

Mistral Small 3.2 IS the current latest (May 2026). 3.1 and 2501 are superseded. **The wiki canonical page is `mistral-small-3.2-24b`.**

## Citation URLs
- [HF: Mistral-Small-24B-Instruct-2501](https://huggingface.co/mistralai/Mistral-Small-24B-Instruct-2501)
- [HF: Mistral-Small-3.1-24B-Instruct-2503](https://huggingface.co/mistralai/Mistral-Small-3.1-24B-Instruct-2503)
- [HF: Mistral-Small-3.2-24B-Instruct-2506](https://huggingface.co/mistralai/Mistral-Small-3.2-24B-Instruct-2506)
- [Mistral Small 3 announcement](https://mistral.ai/news/mistral-small-3)
- [Mistral Small 3.1 announcement](https://mistral.ai/news/mistral-small-3-1)
- [VentureBeat: 3.1 → 3.2 update](https://venturebeat.com/ai/mistral-just-updated-its-open-source-small-model-from-3-1-to-3-2-heres-why)

## Pages updated on ingest
- [[models/mistral-small-24b]] (rewritten to focus on 3.2)
- [[wiki/comparisons/tool-calling-models-on-a10g]]
