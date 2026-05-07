---
tags: [models, tool-calling, specialist, salesforce, non-commercial]
params: 7B (v1) / 8B (v2 Llama)
active_params: same
license: CC-BY-NC-4.0 — non-commercial (all xLAM checkpoints)
context: 4k (xLAM-7b-fc-r) / 32k (xLAM-7b-r) / 128k (Llama-xLAM-2-8b-fc-r)
release_date: 2024-07-17 (v1 fc-r); 2025-04-04 (v2)
last_updated: 2026-05-07
source_count: 3
---

# Salesforce xLAM family

Salesforce's "Large Action Model" specialist line. Strong BFCL performance, but **every Salesforce-released xLAM checkpoint (v1 and v2, fc-r and r, all sizes) is CC-BY-NC-4.0 — non-commercial only.** Commercial use requires Salesforce's closed Platform offering.

For the canonical A10G-fitting xLAM, see **`Salesforce/Llama-xLAM-2-8b-fc-r`** below.

## License — the deployment gotcha

| Variant | License | Commercial? |
|---|---|---|
| xLAM-7b-fc-r | CC-BY-NC-4.0 + DeepSeek model license | **No** |
| xLAM-7b-r | CC-BY-NC-4.0 | **No** |
| Llama-xLAM-2-8b-fc-r | CC-BY-NC-4.0 + Llama 3.1 Community | **No** |
| xLAM-2-32b-fc-r | CC-BY-NC-4.0 | **No** |
| xLAM-2 family (all sizes) | CC-BY-NC-4.0 | **No** |

Llama-xLAM-2 card explicitly states: *"This release is for research purposes only in support of an academic paper."* [Source: [[sources/xlam-family-cards]]]

## Variants and A10G fit

### xLAM-7b-fc-r (v1, July 2024)
- HF: `Salesforce/xLAM-7b-fc-r`
- 6.91B params; base: **DeepSeek-Coder 7B** (not Llama)
- Context: **4K** (very short)
- BFCL **v1** rank 3 at 88.24% (cutoff 2024-07-18) — pre-V3, not directly comparable to current scores
- Fit: ✅ comfortable on A10G (~14 GB FP16, ~4 GB INT4)
- Tool-call format: `{"tool_calls": [{"name": ..., "arguments": {...}}]}` JSON
- vLLM: mainline `xlam` parser is officially scoped to xLAM-2 — v1 likely needs the `xlam_tool_call_parser.py` plugin from the HF repo

### xLAM-7b-r (v1, Sep 2024)
- HF: `Salesforce/xLAM-7b-r` — different from fc-r
- 7.24B params; 32K context
- Card cites BFCL **v2**; specific score is in image, not extractable
- Fit: ✅
- Tool-call format: `{"thought": "...", "tool_calls": [...]}` JSON

### Llama-xLAM-2-8b-fc-r (v2, April 2025) — **the A10G-relevant one**
- HF: `Salesforce/Llama-xLAM-2-8b-fc-r`
- 8B params; base: **Llama 3.1 8B**; context **128K**
- Trained and evaluated on **BFCL v3** + τ-bench (per APIGen-MT paper, [arXiv 2504.03601](https://arxiv.org/abs/2504.03601))
- Specific 8B BFCL score is in chart images on the card; the 70B sibling hits **τ-bench 56.2% pass@1** — beats GPT-4o 52.9%, approaches Claude 3.5 Sonnet 60.1%
- Fit: ✅ comfortable on A10G (~16 GB FP16, ~5 GB INT4)
- Tool-call format: OpenAI-compatible JSON via tokenizer's `apply_chat_template(..., tools=...)`

```
vllm serve Salesforce/Llama-xLAM-2-8b-fc-r \
  --tool-call-parser xlam \
  --chat-template examples/tool_chat_template_xlam_llama.jinja \
  --enable-auto-tool-choice
```

Requires `vllm>=0.6.5`. The card's older `--tool-parser-plugin ./xlam_tool_call_parser.py` route is now redundant on current vLLM. [Source: [[sources/vllm-tool-calling-docs]]]

### Other xLAM-2 sizes
- `Salesforce/xLAM-2-1b-fc-r` (1B), `xLAM-2-3b-fc-r` (3B): too small for serious tool-calling work but useful for edge tests.
- `Salesforce/xLAM-2-32b-fc-r` (33B Qwen 2.5 base): does **NOT** fit single A10G even at AWQ INT4 once KV cache included.
- `Salesforce/xLAM-2-70b-fc-r` (70B Llama base): multi-GPU only.

## Recommendation

- **Commercial deployment**: skip xLAM entirely. Use [[models/hermes-3-llama-3.1-8b]] (Llama community license, commercial-OK) or [[models/granite-3.1-8b]] (Apache-2.0). [Source: [[sources/xlam-family-cards]]]
- **Research / academic use**: `Llama-xLAM-2-8b-fc-r` is the sole xLAM variant combining (a) fits A10G, (b) BFCL v3 evaluation, (c) vLLM mainline support.

## Cost (AWS, on-demand, us-east-1)
- Smallest box: **g5.xlarge** ($1.006/hr) — *research only*. [Source: [[sources/aws-ec2-pricing-2026-05]]]

## Related
- [[models/functionary-small]], [[models/hermes-3-llama-3.1-8b]] — alternative tool-calling specialists
- [[concepts/tool-selection]]
- [[wiki/comparisons/tool-calling-models-on-a10g]]

## Sources
- [[sources/xlam-family-cards]]
- [[sources/vllm-tool-calling-docs]]
- [[sources/bfcl-leaderboard-2026-05]]
