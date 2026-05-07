---
tags: [models, generalist, tool-calling, mistral]
params: 24B
active_params: 24B
license: Apache-2.0
context: 128k (3.1+); 32k (2501)
release_date: 2025-06 (3.2)
last_updated: 2026-05-07
source_count: 3
---

# Mistral-Small-3.2-24B-Instruct (and 3.1 / 2501 predecessors)

Mistral's 24B generalist with explicit "agentic" / tool-calling positioning. **Apache-2.0** (commercially deployable). Useful as a middle ground between 14B-class and 32B-class. **Latest: 3.2 (2506)** with 128K context and a more robust tool-calling template.

- Latest HF: `mistralai/Mistral-Small-3.2-24B-Instruct-2506` ([model card](https://huggingface.co/mistralai/Mistral-Small-3.2-24B-Instruct-2506))
- 3.1: `mistralai/Mistral-Small-3.1-24B-Instruct-2503` (Mar 2025; 128K)
- 3.0: `mistralai/Mistral-Small-24B-Instruct-2501` (Jan 2025; 32K)

## Fit on A10G (24 GB)
- FP16: ~48–55 GB → ❌
- INT8: ~24 GB → ❌ (no headroom)
- AWQ INT4: ~13–14 GB → ✅ ~9 GB KV budget → ~30k ctx at batch=1
- **Verdict**: ✅ at AWQ INT4 — solid fit on g5.xlarge

Common 4-bit quants on HF: `bartowski/...-GGUF`, `unsloth/...-bnb-4bit`, `RedHatAI/...-NVFP4` (NVFP4 won't accelerate on Ampere). [Source: [[sources/mistral-small-3.x-cards]]]

## Strengths (Mistral self-report)

| Benchmark | 3.1 (2503) | 3.2 (2506) |
|---|---|---|
| HumanEval / HumanEval+ pass@5 | 88.4 / 88.99 | — / **92.90** |
| MBPP / MBPP+ pass@5 | 74.7 / 74.63 | — / 78.33 |
| MMLU | 80.62 | 80.50 |
| MMLU-Pro 5-shot CoT | 66.76 | 69.06 |
| MATH | 69.30 | 69.42 |
| GPQA Diamond | 45.96 | 46.13 |
| Wildbench v2 | 55.6 | 65.33 |
| Arena Hard v2 | 19.56 | **43.10** |
| IFEval (internal) | 82.75 | 84.78 |

3.2's headline change vs 3.1: "**more robust function calling template**" for the `mistral` tool parser. [Source: [[sources/mistral-small-3.x-cards]], [[sources/code-benchmarks-2026-05]]]

## Weaknesses
- **No BFCL number published in any official card.** The "agentic" claim is qualitative only. [Source: [[sources/bfcl-leaderboard-2026-05]]]
- **No SWE-bench Verified** or **LiveCodeBench** number.
- Code: HumanEval 88.4 (3.1) / 92.90 HE+ (3.2) — competitive, but trails dedicated code models like [[models/qwen2.5-coder-14b]] on multi-language MultiPL-E.

## vLLM serving notes (3.2 recommended)

```
vllm serve mistralai/Mistral-Small-3.2-24B-Instruct-2506 \
  --tokenizer_mode mistral --config_format mistral --load_format mistral \
  --tool-call-parser mistral --enable-auto-tool-choice
```

Recommended temperature 0.15. Card warns: "Transformers implementation is untested; vLLM recommended for production use." For HF-format checkpoints, swap mistral `tokenizer_mode/config_format/load_format` for `hf`. [Source: [[sources/vllm-tool-calling-docs]]]

## Cost (AWS, on-demand, us-east-1)
- Smallest box: **g5.xlarge** ($1.006/hr)
- For FP16 quality, step to g5.12xlarge with TP=4 ($5.672/hr).
- [Source: [[sources/aws-ec2-pricing-2026-05]]]

## Successor / current state

**3.2 (2506) is the current latest**, May 2026. 3.1 and 2501 are superseded but still available. For production deployments, pin to a specific dated revision to avoid the "Mistral Small versioning is messy" trap.

## Related
- [[models/codestral-22b]] — Mistral's code-specialized sibling (but **MNPL — non-commercial**)
- [[concepts/tool-selection]]
- [[infrastructure/vllm]]
- [[wiki/comparisons/tool-calling-models-on-a10g]]

## Sources
- [[sources/mistral-small-3.x-cards]]
- [[sources/vllm-tool-calling-docs]]
- [[sources/code-benchmarks-2026-05]]
