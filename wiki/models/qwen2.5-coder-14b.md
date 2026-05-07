---
tags: [models, code, generalist, qwen]
params: 14B
active_params: 14B
license: Apache-2.0
context: 32k native, 128k via YaRN
release_date: 2024-11-12
last_updated: 2026-05-07
source_count: 3
---

# Qwen2.5-Coder-14B-Instruct

> **Superseded for new deployments by [[models/qwen3-coder-30b-a3b]]** — Apache-2.0
> MoE 31B/3.3B with mainline `qwen3_coder` parser (fixes the flaky-`hermes`-parser
> issue documented below), SWE-bench Verified ~50%, similar A10G AWQ INT4 fit.

Alibaba's 14B code model. **The current sweet spot** for A10G — much stronger than 7B on real coding tasks while still leaving comfortable KV-cache budget at AWQ INT4.

- HF: `Qwen/Qwen2.5-Coder-14B-Instruct` ([model card](https://huggingface.co/Qwen/Qwen2.5-Coder-14B-Instruct))
- Architecture: 14.7B total / 13.1B non-embedding, dense, 48 layers, 40 Q heads / 8 KV heads (GQA)

## Fit on A10G (24 GB)
- FP16/BF16 weights: ~28 GB → ❌ won't fit
- AWQ INT4: ~8–9 GB → ✅ comfortable; ~14 GB KV-cache budget → ~50k ctx at batch=1
- INT8: ~14 GB → ⚠ tight; ~8 GB for KV cache
- **Verdict**: ✅ at AWQ INT4 — recommended default for code-focused workloads on g5.xlarge

Official AWQ checkpoint: `Qwen/Qwen2.5-Coder-14B-Instruct-AWQ` [Source: [[sources/qwen2.5-coder-tech-report]]]

## Strengths

Code (Qwen2.5-Coder tech report v3, all FP16/BF16):
- HumanEval 89.6 / HumanEval+ 87.2
- MBPP 86.2 / MBPP+ 72.8
- BigCodeBench Complete 81.0 / Hard 22.3 / Instruct 48.4
- LiveCodeBench (07–11/2024) 23.4
- MultiPL-E (Python/Java/C++/TS/JS) 89.0 / 79.7 / 85.1 / 86.8 / 84.5
- CRUXEval Input-CoT 69.5 / Output-CoT 79.5
- Aider Pass@1 58.6 / Pass@2 69.2 (legacy edit board)

[Source: [[sources/qwen2.5-coder-tech-report]], [[sources/code-benchmarks-2026-05]]]

## Weaknesses
- **Not on BFCL leaderboard** — Qwen2.5-Coder family is absent from current BFCL despite tool-calling claims. [Source: [[sources/bfcl-leaderboard-2026-05]]]
- No published **SWE-bench Verified** score.
- Tool-calling not as strong as specialist models like [[models/hermes-3-llama-3.1-8b]] on real workloads.
- General-knowledge tasks behind same-size generalists like [[models/granite-3.1-8b]] (which has Granite 3.3 with HE 89.73).

## vLLM serving notes
- Tool-call parser officially `--tool-call-parser hermes`
- ⚠ **Known parser bug on Coder** (same as 7B variant) — model emits ```json``` fences, not `<tool_call>` tags. vLLM issues #10952, #29192, #32926. Workaround: hanXen/vllm-qwen2.5-coder-tool-parser plugin or community PR #32931. [Source: [[sources/vllm-tool-calling-docs]]]
- Pair with `--enable-prefix-caching` (default in vLLM V1) for tool-heavy workloads.

## Cost (AWS, on-demand, us-east-1)
- Smallest box: **g5.xlarge** ($1.006/hr, ~$734/mo)
- Spot: typically ~$0.30–$0.60/hr
- [Source: [[sources/aws-ec2-pricing-2026-05]]]

## Successor

**Qwen3-Coder family** released February 2026. For new deployments evaluate Qwen3-Coder; Qwen2.5-Coder-14B remains the benchmark-saturated A10G default with the most published numbers.

## Related
- [[models/qwen2.5-coder-7b]], [[models/qwen2.5-coder-32b]]
- [[concepts/code-generation]], [[concepts/tool-selection]]
- [[infrastructure/vllm]], [[infrastructure/quantization]]
- [[wiki/comparisons/tool-calling-models-on-a10g]]

## Sources
- [[sources/qwen2.5-coder-tech-report]]
- [[sources/vllm-tool-calling-docs]]
- [[sources/code-benchmarks-2026-05]]
