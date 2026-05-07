---
tags: [models, tool-calling, specialist, salesforce]
params: 7B
active_params: 7B
license: cc-by-nc-4.0 (non-commercial) — verify per-variant
context: 32k
release_date: 2024-09
last_updated: 2026-05-07
source_count: 0
---

# xLAM-7B (Salesforce)

Salesforce's "xLAM" (large action model) family, fine-tuned specifically for function calling. The 7B variant has historically topped or near-topped BFCL among small models. **License is the catch** — most xLAM checkpoints are non-commercial; some variants (xLAM-2, fc-r) may have different terms — verify.

## Fit on A10G (24 GB)
- FP16: ~14 GB → ✅ ~8 GB KV budget
- AWQ INT4: ~4 GB → ✅ huge KV budget
- **Verdict**: ✅ excellent A10G fit

## Strengths
- **Tool calling: top-tier on BFCL for ≤8B models** *(unverified — needs source)*
- Specialist focus → reliable JSON output, good arg typing
- Compact — among the cheapest models to serve

## Weaknesses
- **Non-commercial license** on most checkpoints — major caveat. Verify the specific variant before any commercial deployment
- General reasoning / code lags generalist 7Bs (specialist tradeoff)
- 32k context (smaller than 128k models)
- Tool-call format may not match a vLLM parser out-of-the-box — verify

## vLLM serving notes
- May need `--tool-call-parser pythonic` or custom chat template depending on variant *(unverified — verify against vLLM docs)*
- Constrained decoding recommended for guaranteed JSON format
- Salesforce maintains AWQ/GPTQ checkpoints on HF for some variants

## Cost (AWS, on-demand, us-east-1)
- Smallest box: **g5.xlarge** (~$1.006/hr)
- For commercial use, prefer [[models/hermes-3-llama-3.1-8b]] or [[models/granite-3.1-8b]]

## Related
- [[models/functionary-small]] — other tool-calling specialist
- [[models/hermes-3-llama-3.1-8b]] — commercial-friendly alternative
- [[concepts/tool-selection]]

## Sources
- (none yet)

## TODO / verify
- HF model cards: Salesforce/xLAM-7b-r, Salesforce/xLAM-7b-fc-r, xLAM-2-* variants
- BFCL leaderboard entry
- License terms per variant
- vLLM parser compatibility
- xLAM technical reports
