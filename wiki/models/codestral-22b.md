---
tags: [models, code, mistral]
params: 22B
active_params: 22B
license: Mistral Non-Production License (MNPL) — non-commercial
context: 32k
release_date: 2024-05
last_updated: 2026-05-07
source_count: 0
---

# Codestral-22B-v0.1

Mistral's code-specialized 22B model. Strong on code completion / fill-in-the-middle, multi-language. **License is the catch**: MNPL is non-commercial — verify your use case before deploying.

## Fit on A10G (24 GB)
- FP16: ~44 GB → ❌
- INT8: ~22 GB → ⚠ no headroom
- AWQ INT4: ~13 GB → ✅ ~9 GB KV budget → ~30k ctx at batch=1
- **Verdict**: ✅ at AWQ INT4 (license permitting)

## Strengths
- Code: strong fill-in-the-middle and completion
- 80+ languages including less-common ones (Bash, Swift, Fortran)
- HumanEval / MBPP: high but **behind Qwen2.5-Coder-14B** *(unverified — needs source)*

## Weaknesses
- **Non-commercial license** (MNPL) — not Apache; check before production use
- Tool calling not the focus — may need constrained decoding
- 32k context

## vLLM serving notes
- `--tool-call-parser mistral` for tool calls *(unverified for Codestral specifically)*
- AWQ/GPTQ widely available

## Cost (AWS, on-demand, us-east-1)
- Smallest box: **g5.xlarge** (~$1.006/hr)
- Note: license restricts commercial use; for commercial code specialists, prefer [[models/qwen2.5-coder-14b]] (Apache-2.0)

## Related
- [[models/qwen2.5-coder-14b]] — Apache-licensed alternative
- [[models/mistral-small-24b]] — generalist sibling
- [[concepts/code-generation]]

## Sources
- (none yet)

## TODO / verify
- HF model card: mistralai/Codestral-22B-v0.1
- MNPL license terms (commercial cutoffs)
- Mistral's Codestral announcement
- Newer Codestral revisions / Codestral Mamba
