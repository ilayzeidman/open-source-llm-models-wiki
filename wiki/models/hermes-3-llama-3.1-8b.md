---
tags: [models, tool-calling, specialist, nous-research]
params: 8B
active_params: 8B
license: Llama 3.1 Community License
context: 128k
release_date: 2024-08
last_updated: 2026-05-07
source_count: 0
---

# Hermes-3-Llama-3.1-8B

NousResearch's tool-calling-focused fine-tune of [[models/llama-3.1-8b-instruct]]. The base for vLLM's `hermes` tool-call parser, also used by Qwen2.5 deployments.

## Fit on A10G (24 GB)
- FP16: ~16 GB → ✅ ~6 GB KV budget
- AWQ INT4: ~5 GB → ✅ huge KV budget
- **Verdict**: ✅ excellent A10G fit

## Strengths
- Tool calling: among the best 8B models on BFCL *(unverified — needs source)*
- Strong system-prompt steerability (a Hermes signature)
- 128k context (inherited from Llama 3.1)
- Large variants exist (Hermes-3-Llama-3.1-70B, Hermes-3-Llama-3.2-405B) for multi-GPU upgrades

## Weaknesses
- Code: not a code specialist; prefer [[models/qwen2.5-coder-7b]] for code-heavy work
- Inherits Llama license (community license, not Apache)

## vLLM serving notes
- Tool-call parser: `--tool-call-parser hermes` (this model is the canonical target for that parser)
- Uses XML-ish `<tool_call>...</tool_call>` blocks — see [[concepts/tool-selection]]
- AWQ/GPTQ widely available

## Cost (AWS, on-demand, us-east-1)
- Smallest box: **g5.xlarge** (~$1.006/hr)

## Related
- [[models/llama-3.1-8b-instruct]] — base model
- [[models/xlam-7b]], [[models/functionary-small]] — other tool-calling specialists
- [[concepts/tool-selection]], [[infrastructure/vllm]]

## Sources
- (none yet)

## TODO / verify
- HF model card: NousResearch/Hermes-3-Llama-3.1-8B
- BFCL leaderboard entry
- Hermes-3 technical report / blog
- Whether Hermes-4 / newer revisions exist
