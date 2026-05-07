---
tags: [models, generalist, tool-calling, ibm]
params: 8B
active_params: 8B
license: Apache-2.0
context: 128k
release_date: 2024-12
last_updated: 2026-05-07
source_count: 0
---

# Granite-3.1-8B-Instruct

IBM's 8B generalist with explicit first-class tool-calling support and a dedicated vLLM parser. Apache-2.0. Less hyped than Llama/Qwen but a credible enterprise-friendly default.

## Fit on A10G (24 GB)
- FP16: ~16 GB → ✅ ~6 GB KV budget
- AWQ INT4: ~5 GB → ✅ ~17 GB KV budget — long context feasible
- **Verdict**: ✅ comfortable on g5.xlarge

## Strengths
- Tool calling: native, with `granite` parser in vLLM
- Apache-2.0, enterprise-positioned
- 128k context
- BFCL: solid for an 8B *(unverified — needs source)*

## Weaknesses
- Code: weaker than dedicated code 7Bs like [[models/qwen2.5-coder-7b]] *(unverified)*
- Less community tooling/finetune ecosystem than Llama/Qwen

## vLLM serving notes
- Tool-call parser: `--tool-call-parser granite`
- AWQ/GPTQ checkpoints available

## Cost (AWS, on-demand, us-east-1)
- Smallest box: **g5.xlarge** (~$1.006/hr)
- Among the cheapest tool-calling-capable models to serve

## Related
- [[models/llama-3.1-8b-instruct]] — comparable size, different family
- [[models/hermes-3-llama-3.1-8b]] — alt 8B with strong tool calling
- [[concepts/tool-selection]]

## Sources
- (none yet)

## TODO / verify
- HF model card: ibm-granite/granite-3.1-8b-instruct
- Granite 3.1 announcement / technical report
- BFCL score
- Granite 3.2/3.3 if released and superseding
