---
tags: [models, code, mistral, non-commercial]
params: 22B
active_params: 22B
license: MNPL-0.1 — non-commercial
context: 32k
release_date: 2024-05-29
last_updated: 2026-05-07
source_count: 3
---

# Codestral-22B-v0.1

Mistral's code-specialized 22B model. Strong on code completion / fill-in-the-middle, multi-language. **License is the catch**: Mistral AI Non-Production License (MNPL) — strictly non-commercial. Commercial use requires a paid agreement with Mistral sales.

- HF: `mistralai/Codestral-22B-v0.1` ([model card](https://huggingface.co/mistralai/Codestral-22B-v0.1))

## Fit on A10G (24 GB)
- FP16: ~44 GB → ❌
- INT8: ~22 GB → ⚠ no headroom
- AWQ INT4: ~13 GB → ✅ ~9 GB KV budget → ~30k ctx at batch=1
- **Verdict**: ✅ at AWQ INT4 — *license permitting*

50+ community quantizations on HF; **no first-party AWQ/GPTQ from `mistralai/`** (consistent with the MNPL stance). [Source: [[sources/codestral-mnpl-license]]]

## Strengths (Mistral announcement)

- HumanEval (Python) **86.6**
- MBPP (sanitized) **91.2**
- HumanEval-X across 80+ languages — exact per-language values not preserved on the live page
- Strong fill-in-the-middle and code completion
- Aider Ollama Q8 (`codestral:22b-v0.1-q8_0`): 35.3 / 48.1 pass@1/pass@2
- Aider codestral-2405 (API): 35.3 / 51.1

[Source: [[sources/codestral-mnpl-license]], [[sources/code-benchmarks-2026-05]]]

## Weaknesses
- **Non-commercial license** (MNPL-0.1). SaaS, hosted services, and any paid-or-free commercial supply are prohibited. [Source: [[sources/codestral-mnpl-license]]]
- **Not trained on tool-call format.** `--tool-call-parser mistral` exists but is tuned for Mistral-Instruct-v0.3+ and Mistral Small — treat tool calls as unsupported on Codestral.
- 32k context (smaller than 128k models).
- **No published BFCL, SWE-bench Verified, or LiveCodeBench** scores.
- Card warns: "no moderation mechanisms."

## vLLM serving notes
- Mistral `[INST]…[/INST]` template + FIM tokens.
- `--tool-call-parser mistral` exists but is not Codestral-trained — use constrained decoding only if tool calls are required.
- Many community AWQ/GGUF available for self-deployment.
- [Source: [[sources/vllm-tool-calling-docs]]]

## Cost (AWS, on-demand, us-east-1)
- Smallest box: **g5.xlarge** ($1.006/hr) — but again, MNPL gates this for any commercial use.
- For commercial code specialists, prefer [[models/qwen2.5-coder-14b]] (Apache-2.0) or **Codestral Mamba 7B** (Apache-2.0). [Source: [[sources/aws-ec2-pricing-2026-05]]]

## Commercial-friendly Mistral alternatives

- **Codestral Mamba 7B** (`mistralai/Mamba-Codestral-7B-v0.1`) — **Apache-2.0**, Mamba2 architecture, July 2024.
- **Codestral 25.01 / Codestral V2** — Jan 2025, ~<100B params, 256K context, **NOT open-weight** (API/IDE only). Not deployable.
- **Devstral** family — targets later SWE-agent use cases.

[Source: [[sources/codestral-mnpl-license]]]

## Related
- [[models/qwen2.5-coder-14b]] — Apache-licensed alternative
- [[models/mistral-small-24b]] — Apache-licensed generalist sibling
- [[concepts/code-generation]]
- [[wiki/comparisons/tool-calling-models-on-a10g]]

## Sources
- [[sources/codestral-mnpl-license]]
- [[sources/code-benchmarks-2026-05]]
- [[sources/vllm-tool-calling-docs]]
