---
tags: [models, code, generalist, qwen]
params: 7B
active_params: 7B
license: Apache-2.0
context: 32k native, 128k via YaRN
release_date: 2024-11-12
last_updated: 2026-05-07
source_count: 3
---

# Qwen2.5-Coder-7B-Instruct

Alibaba's Qwen team's 7B code model. Strong code reasoning for its size, native 32K (128K with YaRN) context, decent tool calling. Sweet spot for low-cost A10G deployment when 14B-class quality isn't needed.

- HF: `Qwen/Qwen2.5-Coder-7B-Instruct` ([model card](https://huggingface.co/Qwen/Qwen2.5-Coder-7B-Instruct))
- Architecture: 7.61B total / 6.53B non-embedding, dense, GQA

## Fit on A10G (24 GB)
- FP16/BF16 weights: ~14 GB → ✅ fits with ~9 GB KV-cache budget
- AWQ INT4: ~5 GB → ✅ huge KV-cache budget (~17 GB) → 100k+ ctx feasible at batch=1
- **Verdict**: ✅ comfortable on g5.xlarge

Official AWQ checkpoint: `Qwen/Qwen2.5-Coder-7B-Instruct-AWQ` [Source: [[sources/qwen2.5-coder-tech-report]]]

## Strengths

Code (Qwen2.5-Coder tech report v3, all FP16/BF16):
- HumanEval 88.4 / HumanEval+ 84.1
- MBPP 83.5 / MBPP+ 71.7
- BigCodeBench Complete 76.9 / Hard 16.2 / Instruct 41.0
- LiveCodeBench (07–11/2024) 18.2
- MultiPL-E (Python/Java/C++/TS/JS) 87.8 / 76.5 / 75.6 / 81.8 / 83.2
- CRUXEval Input-CoT 65.8 / Output-CoT 65.9
- Aider Pass@1 55.6 / Pass@2 68.4

[Source: [[sources/qwen2.5-coder-tech-report]], [[sources/code-benchmarks-2026-05]]]

## Weaknesses
- **Not on BFCL leaderboard** — no published BFCL number despite Qwen team's tool-calling claims. [Source: [[sources/bfcl-leaderboard-2026-05]]]
- **No SWE-bench Verified** number published.
- General reasoning weaker than 14B+ models.

## vLLM serving notes
- Tool-call parser officially `--tool-call-parser hermes`
- ⚠ **Known parser bug on Coder**: emits ```json``` fenced output instead of `<tool_call>` tags; the `hermes` parser mis-extracts. vLLM issues #10952, #29192, #32926. Workaround: hanXen/vllm-qwen2.5-coder-tool-parser plugin. [Source: [[sources/vllm-tool-calling-docs]]]
- ChatML chat template; vLLM supports static YaRN only — set `rope_scaling` only when long context needed.
- Pre-quantized AWQ + GPTQ-Int4 + GPTQ-Int8 published officially by Qwen team.

## Cost (AWS, on-demand, us-east-1)
- Smallest box: **g5.xlarge** ($1.006/hr, ~$734/mo)
- Spot: typically ~$0.30–$0.60/hr
- Highly throughput-friendly at AWQ INT4 — among the cheapest models to serve. [Source: [[sources/aws-ec2-pricing-2026-05]]]

## Successor

**Qwen3-Coder family** released February 2026. Qwen3-Coder-Next claims significantly higher SWE-bench Verified. For new deployments, evaluate Qwen3-Coder; Qwen2.5-Coder remains a strong stable baseline with full published benchmarks.

## Related
- [[models/qwen2.5-coder-14b]], [[models/qwen2.5-coder-32b]]
- [[concepts/code-generation]], [[concepts/tool-selection]]
- [[infrastructure/vllm]], [[hardware/a10g-g5xlarge]]
- [[wiki/comparisons/tool-calling-models-on-a10g]]

## Sources
- [[sources/qwen2.5-coder-tech-report]]
- [[sources/vllm-tool-calling-docs]]
- [[sources/code-benchmarks-2026-05]]
