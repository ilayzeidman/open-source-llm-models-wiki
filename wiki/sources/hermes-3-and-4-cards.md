---
tags: [source, models, tool-calling, nous-research]
source_path: raw/hermes-3-llama-3.1-8b-hf-card.md, raw/hermes-4-70b-hf-card.md
source_url: https://huggingface.co/NousResearch/Hermes-3-Llama-3.1-8B
ingested: 2026-05-07
last_updated: 2026-05-07
---

# NousResearch Hermes-3 / Hermes-4 cards

## Key claims

### Hermes-3-Llama-3.1-8B
- **HF model ID**: `NousResearch/Hermes-3-Llama-3.1-8B` (also 70B / 405B variants; GGUF mirror exists)
- **Params**: 8B (BF16). Base: `meta-llama/Llama-3.1-8B`.
- **Context**: 128K (Llama 3.1 base)
- **Released**: 2024-08-15 (arXiv 2408.11857)
- **License**: Llama 3.1 Community License (commercial-OK with the same 700M MAU + AUP + "Built with Llama" attribution clauses; see [[sources/llama-3.x-cards]])

#### Tool-call format
ChatML system prompt with `<tools>...</tools>` schema injection; assistant emits `<tool_call>{"name": ..., "arguments": ...}</tool_call>` JSON-in-XML. **Canonical "Hermes format"** — this is the model that the vLLM `hermes` parser is canonically tuned for.

#### Benchmarks (limited — Hermes-3 8B is sparsely benchmarked)
- **No official BFCL number on the card.**
- Community blogs report ~91% but this is the well-formed-JSON rate, **not** BFCL overall accuracy. Don't quote as comparable.
- Open LLM Leaderboard avg 23.49 (IFEval 61.70, BBH 30.72, MMLU-PRO 23.77).

#### vLLM serving
- First-class: `--tool-call-parser hermes`
- vLLM docs explicitly say "all Nous Hermes models newer than Hermes 2 Pro are supported." Hermes 2 Theta is flagged as degraded — avoid.

### Hermes-4-70B / 405B
- **HF model IDs**: `NousResearch/Hermes-4-70B`, `NousResearch/Hermes-4-405B`, `NousResearch/Hermes-4-405B-FP8`
- **Note: NO 8B variant.** The 8B gap remains for A10G-class deployment — Hermes-3-Llama-3.1-8B is still the only Nous specialist that fits.
- **Params**: 70B / 405B. Base: Llama 3.1 70B / 405B.
- **Context**: 128K (Llama 3.1 inheritance)
- **Released**: 2025-08-25 (arXiv 2508.18255)
- **License**: Llama 3.1 Community License — commercial-OK with the same caveats.

#### Tool-call format
Same `<tool_call>{...}</tool_call>` JSON-in-XML; *new*: hybrid reasoning mode emits `<think>...</think>` before the answer when activated.

#### vLLM serving
- Card explicitly endorses `vllm --tool-call-parser hermes`. SGLang uses `qwen25` parser.

### Deployment note (A10G)
- Hermes-3-Llama-3.1-8B fits comfortably on a single A10G (AWQ INT4 ~5 GB).
- Hermes-4-70B and 405B do not fit single A10G in any quant. Need 4× A10G (g5.12xlarge) or step to A100/H100.

## Citation URLs
- [HF: Hermes-3-Llama-3.1-8B](https://huggingface.co/NousResearch/Hermes-3-Llama-3.1-8B)
- [HF: Hermes-4-70B](https://huggingface.co/NousResearch/Hermes-4-70B)
- [HF: Hermes-4-405B](https://huggingface.co/NousResearch/Hermes-4-405B)
- [Hermes-3 Tech Report (arXiv 2408.11857)](https://arxiv.org/abs/2408.11857)
- [Hermes-Function-Calling repo](https://github.com/NousResearch/Hermes-Function-Calling)

## Pages updated on ingest
- [[models/hermes-3-llama-3.1-8b]]
- [[wiki/comparisons/tool-calling-models-on-a10g]]
