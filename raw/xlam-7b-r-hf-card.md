# Salesforce/xLAM-7b-r — Hugging Face model card

source_url: https://huggingface.co/Salesforce/xLAM-7b-r
fetched: 2026-05-07

## Identity

- HF model ID: `Salesforce/xLAM-7b-r`
- Params: 7.24B
- Context: **32k tokens**
- Released: Sep 5, 2024
- License: **CC-BY-NC-4.0** (research-only)
- Card: "released exclusively for research purposes" + "A new and enhanced version of xLAM will soon be available exclusively to customers on our Platform" — i.e. commercial use of this checkpoint is not permitted.

## Position in the family

- "General tool use series" (vs the specialized `fc` series).
- Larger context window (32k vs 4k for `xLAM-7b-fc-r`).
- 7.24B vs 6.91B for the fc variant.

## Tool-call output format

```json
{
  "thought": "the thought process, or an empty string",
  "tool_calls": [
    {
      "name": "api_name1",
      "arguments": {
        "argument1": "value1",
        "argument2": "value2"
      }
    }
  ]
}
```

Card describes this as similar to OpenAI function-calling.

## BFCL

- Card references BFCL **v2** leaderboard (cutoff 09/03/2024)
- Specific score not in extracted text (image-based table on the card)

## Other variants in the v1 "r" series

- `xLAM-8x7b-r` — 46.7B params, 32k ctx
- `xLAM-8x22b-r` — 141B params, 64k ctx
