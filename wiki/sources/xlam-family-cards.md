---
tags: [source, models, tool-calling, salesforce, license]
source_path: raw/xlam-7b-r-hf-card.md, raw/xlam-7b-fc-r-hf-card.md, raw/llama-xlam-2-8b-fc-r-hf-card.md, raw/xlam-2-32b-fc-r-hf-card.md, raw/cc-by-nc-4.0-terms.md
source_url: https://huggingface.co/Salesforce
ingested: 2026-05-07
last_updated: 2026-05-07
---

# Salesforce xLAM family — model cards & licenses

Salesforce's "Large Action Model" family. **Critical license fact: every Salesforce-released xLAM checkpoint (v1 and v2, fc-r and r, all sizes) is CC-BY-NC-4.0 — non-commercial only.**

## Key claims

### License — the deployment gotcha

| Variant | License | Commercial? |
|---|---|---|
| xLAM-7b-fc-r | CC-BY-NC-4.0 + DeepSeek model license | **No** |
| xLAM-7b-r | CC-BY-NC-4.0 | **No** |
| Llama-xLAM-2-8b-fc-r | CC-BY-NC-4.0 + Llama 3.1 Community | **No** |
| xLAM-2-32b-fc-r | CC-BY-NC-4.0 | **No** |
| xLAM-2 family (all sizes) | CC-BY-NC-4.0 | **No** |

Llama-xLAM-2 card explicitly states: *"This release is for research purposes only in support of an academic paper."* Salesforce's commercial path is their closed Platform offering — not these checkpoints.

### Variants

#### xLAM-7b-fc-r (v1, July 2024)
- HF: `Salesforce/xLAM-7b-fc-r`
- 6.91B params; base = **DeepSeek-Coder 7B** (not Llama)
- Context: **4K** (very short)
- BFCL **v1** rank 3 at 88.24% — pre-V3, not directly comparable to current scores
- Tool-call format: `{"tool_calls": [{"name": ..., "arguments": {...}}]}` JSON
- vLLM mainline `xlam` parser is officially scoped to xLAM-2 — v1 likely needs the `xlam_tool_call_parser.py` plugin from the HF repo.

#### xLAM-7b-r (v1, Sep 2024)
- HF: `Salesforce/xLAM-7b-r` — different from fc-r
- 7.24B params; 32K context
- Card cites BFCL **v2**; specific score is in image, not extractable
- Tool-call format: `{"thought": "...", "tool_calls": [...]}` JSON

#### Llama-xLAM-2-8b-fc-r (v2, April 2025) — the A10G-relevant one
- HF: `Salesforce/Llama-xLAM-2-8b-fc-r`
- 8B params; base = **Llama 3.1 8B**; context 128K
- Trained and evaluated on **BFCL v3** + τ-bench (per APIGen-MT paper)
- Specific 8B score is in chart images on the card (not extractable)
- The 70B sibling hits τ-bench 56.2% pass@1 — beats GPT-4o 52.9%, approaches Claude 3.5 Sonnet 60.1%
- Tool-call format: OpenAI-compatible JSON via tokenizer's `apply_chat_template(..., tools=...)`

##### vLLM serving
```
vllm serve Salesforce/Llama-xLAM-2-8b-fc-r \
  --tool-call-parser xlam \
  --chat-template examples/tool_chat_template_xlam_llama.jinja \
  --enable-auto-tool-choice
```
Requires `vllm>=0.6.5`. The card's older `--tool-parser-plugin ./xlam_tool_call_parser.py` route is now redundant on current vLLM.

#### xLAM-2 sizes
- `Salesforce/xLAM-2-1b-fc-r` (1B)
- `Salesforce/xLAM-2-3b-fc-r` (3B)
- `Salesforce/Llama-xLAM-2-8b-fc-r` (8B, Llama base) — single-A10G fit
- `Salesforce/xLAM-2-32b-fc-r` (33B, Qwen 2.5 base) — does NOT fit single A10G even at AWQ INT4 once KV cache included
- `Salesforce/xLAM-2-70b-fc-r` (70B, Llama base) — multi-GPU only

### Deployment recommendation for A10G research wiki

- **For commercial deployment: skip xLAM entirely.** Use [[models/hermes-3-llama-3.1-8b]] or [[models/granite-3.1-8b]] (both commercial-friendly tool callers).
- **For research / academic use: Llama-xLAM-2-8b-fc-r** is the sole xLAM variant that combines (a) fits A10G, (b) BFCL v3 evaluation, (c) vLLM mainline support.

## Citation URLs
- [HF: Salesforce/xLAM-7b-fc-r](https://huggingface.co/Salesforce/xLAM-7b-fc-r)
- [HF: Salesforce/xLAM-7b-r](https://huggingface.co/Salesforce/xLAM-7b-r)
- [HF: Salesforce/Llama-xLAM-2-8b-fc-r](https://huggingface.co/Salesforce/Llama-xLAM-2-8b-fc-r)
- [HF: Salesforce/xLAM-2-32b-fc-r](https://huggingface.co/Salesforce/xLAM-2-32b-fc-r)
- [APIGen-MT paper (arXiv 2504.03601)](https://arxiv.org/abs/2504.03601)
- [SalesforceAIResearch/xLAM repo](https://github.com/SalesforceAIResearch/xLAM)
- [CC-BY-NC-4.0 legal code](https://creativecommons.org/licenses/by-nc/4.0/legalcode.en)

## Pages updated on ingest
- [[models/xlam-7b]] (rewritten)
- [[wiki/comparisons/tool-calling-models-on-a10g]]
