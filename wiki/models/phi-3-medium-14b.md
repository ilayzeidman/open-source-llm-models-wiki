---
tags: [models, generalist, microsoft]
params: 14B
active_params: 14B
license: MIT
context: 128k
release_date: 2024-05
last_updated: 2026-05-07
source_count: 0
---

# Phi-3-Medium-128k-Instruct

Microsoft's 14B "Phi-3" generalist trained on heavily curated synthetic + filtered web data. **MIT license** is unusually permissive. Strong on reasoning per parameter; tool calling is not a primary focus.

## Fit on A10G (24 GB)
- FP16: ~28 GB → ❌
- INT8: ~14 GB → ✅ ~8 GB KV budget
- AWQ INT4: ~8–9 GB → ✅ comfortable, ~14 GB KV budget
- **Verdict**: ✅ at AWQ INT4

## Strengths
- Reasoning: strong for 14B (Phi family's signature)
- MIT license
- 128k context

## Weaknesses
- Tool calling: weaker than Llama / Mistral / Granite at the same size *(unverified)*
- Code: behind dedicated code models
- Quality occasionally fragile to prompt format (Phi quirk)
- Successor models (Phi-4 family) may already supersede this for many use cases — check current state

## vLLM serving notes
- Phi-3 supported in vLLM; chat template must match the model's expected format
- No dedicated tool-call parser — rely on constrained decoding for JSON tool calls

## Cost (AWS, on-demand, us-east-1)
- Smallest box: **g5.xlarge** (~$1.006/hr)

## Related
- [[concepts/tool-selection]] — use constrained decoding for reliability
- [[infrastructure/vllm]]
- Successors: Phi-4 / Phi-4-mini *(check if these have superseded Phi-3 for this use case)*

## Sources
- (none yet)

## TODO / verify
- HF model card: microsoft/Phi-3-medium-128k-instruct
- Phi-3 / Phi-4 technical reports
- Whether Phi-4 family is now the better choice (likely)
- BFCL score (if any)
