---
tags: [source, models, ibm, granite]
source_path: raw/granite-3.0-announcement.md, raw/granite-3.1-8b-hf-card.md, raw/granite-3.3-8b-hf-card.md
source_url: https://huggingface.co/ibm-granite/granite-3.3-8b-instruct
ingested: 2026-05-07
last_updated: 2026-05-07
---

# IBM Granite 3.x — model cards & announcements

IBM's enterprise-focused 8B generalist with first-class tool calling. **Latest: 3.3.**

## Key claims

### Versions

| Version | HF ID | Released |
|---|---|---|
| 3.0 | `ibm-granite/granite-3.0-8b-instruct` | late 2024 |
| 3.1 | `ibm-granite/granite-3.1-8b-instruct` | 2024-12-18 |
| 3.2 | `ibm-granite/granite-3.2-8b-instruct` | 2025-02 |
| 3.3 | `ibm-granite/granite-3.3-8b-instruct` | 2025-04-16 |
| 4.x | (new architecture line) | 2025+ |

All Granite 3.x are 8.1B dense (40 layers, 4096 embed, 32 heads / 8 KV heads, MLP 12800, SwiGLU, RoPE), 128K context, **Apache-2.0**. IBM offers "uncapped indemnity for third-party IP claims" per the Granite 3.0 announcement.

### Benchmarks

| Benchmark | 3.1-8B | 3.3-8B |
|---|---|---|
| HumanEval | not on card | **89.73** |
| HumanEval+ | not on card | **86.09** |
| IFEval | 72.08 | 74.82 |
| MMLU | 65.34 | 65.54 |
| MATH-500 | — | 69.02 |
| GSM8K | 73.84 | 80.89 |
| Arena-Hard | — | 57.56 |
| AlpacaEval-2.0 | — | 62.68 |
| AIME24 | — | 8.12 |

IBM's 3.0 announcement claims best-in-weight-class **BFCL** scores but no number is published in the announcement itself. Granite 3.1's model card omits BFCL/HumanEval/MBPP entirely.

**Granite 4.1-8B** (newer line) reports BFCL v3 = 68.27 per IBM blog and HumanEval ~87.2.

### vLLM serving
- `--tool-call-parser granite` for both 3.1 and 3.3.
- 3.0 needed an explicit `--chat-template tool_chat_template_granite.jinja`; 3.1+ ship the right chat template in the tokenizer.
- Granite 3.3 also adds `<think>...</think>` / `<response>...</response>` reasoning tags toggled via `tokenizer.apply_chat_template(..., thinking=True)`.
- Granite 4.x uses `--tool-call-parser granite4`.
- 29–30 community quant variants per version (llama.cpp / LM Studio / Jan / Ollama). NVIDIA NIM hosts Granite 3.3.

### Successor / current state

**Granite 3.3 supersedes 3.1** for tool calling: big code-gen jump (HumanEval +24 points), better IFEval, native thinking mode. The wiki canonical IBM 8B page should be `granite-3.3-8b`; keep 3.1 as a historical reference.

## Citation URLs
- [HF: granite-3.1-8b-instruct](https://huggingface.co/ibm-granite/granite-3.1-8b-instruct)
- [HF: granite-3.3-8b-instruct](https://huggingface.co/ibm-granite/granite-3.3-8b-instruct)
- [IBM Granite 3.0 announcement](https://www.ibm.com/new/announcements/ibm-granite-3-0-open-state-of-the-art-enterprise-models)
- [vLLM tool-calling docs](https://docs.vllm.ai/en/latest/features/tool_calling/)

## Pages updated on ingest
- [[models/granite-3.1-8b]] (updated; references 3.3 successor)
- [[wiki/comparisons/tool-calling-models-on-a10g]]
