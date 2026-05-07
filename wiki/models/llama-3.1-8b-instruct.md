---
tags: [models, generalist, tool-calling, meta]
params: 8B
active_params: 8B
license: Llama 3.1 Community License
context: 128k
release_date: 2024-07
last_updated: 2026-05-07
source_count: 0
---

# Llama-3.1-8B-Instruct

Meta's 8B generalist with first-class tool-calling support. Widely deployed baseline; strong vLLM integration. Not a code specialist but solid all-around.

## Fit on A10G (24 GB)
- FP16/BF16 weights: ~16 GB → ✅ fits with ~6 GB KV-cache budget; OK for short context
- AWQ INT4: ~5 GB → ✅ huge KV budget; 100k+ ctx feasible at batch=1
- INT8: ~8 GB → ✅ comfortable
- **Verdict**: ✅ ideal A10G citizen

## Strengths
- Tool calling: native, `llama3_json` parser well-supported in vLLM
- BFCL: **mid-80s** *(unverified — needs source)*
- Solid general reasoning, math, multilingual
- 128k context

## Weaknesses
- Code: **HumanEval ~72** *(unverified)* — well behind code specialists like [[models/qwen2.5-coder-7b]]
- Llama license (community license, not Apache) — has commercial restrictions

## vLLM serving notes
- Tool-call parser: `--tool-call-parser llama3_json`
- Chat template ships with the model — `--chat-template` usually not needed
- Pre-quantized AWQ/GPTQ widely available

## Cost (AWS, on-demand, us-east-1)
- Smallest box: **g5.xlarge** (~$1.006/hr, ~$734/mo)
- High throughput at INT4 — among the cheapest models to serve

## Related
- [[models/llama-3.3-70b-instruct]] — bigger sibling, multi-GPU only
- [[models/hermes-3-llama-3.1-8b]] — tool-calling fine-tune of this base
- [[concepts/tool-selection]], [[infrastructure/vllm]]

## Sources
- (none yet)

## TODO / verify
- HF model card: meta-llama/Meta-Llama-3.1-8B-Instruct
- Llama 3.1 paper / blog
- BFCL leaderboard entry
- License details (commercial use cutoffs)
