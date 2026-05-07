---
tags: [models, tool-calling, specialist, meetkai]
params: 8B
active_params: 8B
license: MIT
context: 128k
release_date: 2024
last_updated: 2026-05-07
source_count: 0
---

# Functionary-Small (MeetKai)

MeetKai's tool-calling specialist family. The "small" variants are typically 7–8B and Llama-derived. Notable for explicit support of **parallel** tool calls and OpenAI-compatible function-calling format. MIT license on most variants — commercial-friendly.

## Fit on A10G (24 GB)
- FP16: ~16 GB → ✅ ~6 GB KV
- AWQ INT4: ~5 GB → ✅ huge KV budget
- **Verdict**: ✅ comfortable on g5.xlarge

## Strengths
- Tool calling: dedicated focus, OpenAI-compatible output
- Parallel tool calls supported
- MIT license (most variants) — commercial-friendly
- 128k context

## Weaknesses
- Less hyped / smaller community than Hermes / xLAM
- Code, general reasoning behind specialist baselines

## vLLM serving notes
- MeetKai has historically maintained their own vLLM fork or PR for full Functionary support; check whether mainline vLLM supports the variant you pick *(unverified — verify against latest vLLM)*
- May need MeetKai's tool-template or a custom chat template

## Cost (AWS, on-demand, us-east-1)
- Smallest box: **g5.xlarge** (~$1.006/hr)

## Related
- [[models/xlam-7b]], [[models/hermes-3-llama-3.1-8b]] — other tool-calling specialists
- [[concepts/tool-selection]]
- [[infrastructure/vllm]]

## Sources
- (none yet)

## TODO / verify
- HF model card: meetkai/functionary-small-v3.x or current variant
- vLLM mainline support status for current Functionary
- BFCL score
- Latest variant naming / version
