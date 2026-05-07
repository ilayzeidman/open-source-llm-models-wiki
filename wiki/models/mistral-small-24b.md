---
tags: [models, generalist, tool-calling, mistral]
params: 24B
active_params: 24B
license: Apache-2.0
context: 32k
release_date: 2025-01
last_updated: 2026-05-07
source_count: 0
---

# Mistral-Small-3-24B-Instruct

Mistral's 24B generalist with explicit "agentic" / tool-calling positioning at release. Apache-2.0. Useful as a middle ground between 14B-class and 32B-class.

## Fit on A10G (24 GB)
- FP16: ~48 GB → ❌
- INT8: ~24 GB → ❌ (no headroom)
- AWQ INT4: ~13–14 GB → ✅ ~9 GB KV budget → ~30k ctx at batch=1
- **Verdict**: ✅ at AWQ INT4 — solid fit on g5.xlarge

## Strengths
- Tool calling: native, well-supported parser in vLLM
- General reasoning competitive with 30B-class
- Apache-2.0 license
- BFCL: **mid-80s** *(unverified — needs source)*

## Weaknesses
- Code: weaker than dedicated code models like [[models/qwen2.5-coder-14b]] *(unverified)*
- 32k context (smaller than the 128k-class models in this wiki)

## vLLM serving notes
- Tool-call parser: `--tool-call-parser mistral` (with appropriate chat template)
- AWQ checkpoints available; some recommend GPTQ for Mistral models *(unverified)*

## Cost (AWS, on-demand, us-east-1)
- Smallest box: **g5.xlarge** (~$1.006/hr)
- For FP16 quality, step to g5.12xlarge with TP=4

## Related
- [[models/codestral-22b]] — Mistral's code-specialized sibling
- [[concepts/tool-selection]]
- [[infrastructure/vllm]]

## Sources
- (none yet)

## TODO / verify
- HF model card: mistralai/Mistral-Small-24B-Instruct-2501 (or current variant)
- Mistral's announcement post for Mistral Small 3
- BFCL leaderboard entry
- vLLM Mistral parser current behavior
