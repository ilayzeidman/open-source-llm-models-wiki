---
tags: [models, code, generalist, qwen]
params: 32B
active_params: 32B
license: Apache-2.0
context: 32k native, 128k via YaRN
release_date: 2024-11-12
last_updated: 2026-05-07
source_count: 3
---

# Qwen2.5-Coder-32B-Instruct

> **Superseded for new deployments by [[models/qwen3-coder-30b-a3b]]** (MoE) or
> **[[models/qwen3-32b]]** (dense, hybrid reasoning), both with mainline
> `qwen3_coder` / `hermes` parsers. Qwen2.5-Coder-32B retains the highest published
> HumanEval (92.7) of the family but is no longer the default — see
> [[hardware/g6e-l40s]] for fitting full FP16 on a single L40S.

Alibaba's flagship open-source code model at release. Reportedly competitive with frontier closed models on code tasks. **Tight fit on a single A10G** even at INT4.

- HF: `Qwen/Qwen2.5-Coder-32B-Instruct` ([model card](https://huggingface.co/Qwen/Qwen2.5-Coder-32B-Instruct))
- Architecture: 32.5B total / 31.0B non-embedding, dense, 64 layers, 40 Q heads / 8 KV heads (GQA)

## Fit on A10G (24 GB)
- FP16/BF16 weights: ~65 GB → ❌ won't fit
- INT8: ~32 GB → ❌ won't fit
- AWQ INT4: ~18–19 GB → ⚠ tight; only ~3 GB KV-cache budget → ~7k effective ctx at batch=1
- **Verdict**: ⚠ runs but with limited context. For real use, prefer [[hardware/multi-gpu-options|g5.12xlarge with TP=2 or TP=4]].

Official AWQ checkpoint: `Qwen/Qwen2.5-Coder-32B-Instruct-AWQ` (1.13M downloads/month — clearly the canonical single-A10G choice). [Source: [[sources/qwen2.5-coder-tech-report]]]

## Strengths

Code (Qwen2.5-Coder tech report v3, all FP16/BF16):
- HumanEval **92.7** / HumanEval+ 87.2
- MBPP 90.2 / MBPP+ 75.1
- BigCodeBench Complete 83.0 / Hard 26.4 / Instruct 49.6
- LiveCodeBench (07–11/2024) **31.4** (later cuts cited up to 51.2)
- MultiPL-E (Python/Java/C++/TS/JS) 92.7 / 80.4 / 79.5 / 86.8 / 85.7
- CRUXEval Input-CoT 75.2 / Output-CoT 83.4
- Aider Pass@1 60.9 / Pass@2 73.7 (legacy edit board)
- Aider polyglot: 16.4% (whole) / 8.0% (diff)
- Aider Ollama Q8: 72.9% pass@2 (the **only quantized public Aider score** for this model)
- McEval 65.9; MdEval 75.2 (Qwen blog)

[Source: [[sources/qwen2.5-coder-tech-report]], [[sources/code-benchmarks-2026-05]]]

## Weaknesses
- **Not on BFCL leaderboard.** No published BFCL score. [Source: [[sources/bfcl-leaderboard-2026-05]]]
- **No SWE-bench Verified** score published — the most-asked-for number for "real engineering" capability is missing.
- A10G fit is tight — context-bound at AWQ INT4.
- Best deployment is multi-GPU; single-A10G is a compromise.

## vLLM serving notes
- `--tool-call-parser hermes` (with the Coder parser caveats — see [[infrastructure/vllm]])
- Set `--max-model-len ~8192` and `--gpu-memory-utilization 0.95` for single-A10G to maximize KV cache
- Consider TP=2 on g5.12xlarge for FP16-quality + comfortable context. [Source: [[sources/vllm-tool-calling-docs]]]

## Cost (AWS, on-demand, us-east-1)
- Single-A10G (compromise): **g5.xlarge** $1.006/hr
- Recommended (full quality, comfortable ctx): **g5.12xlarge** $5.672/hr (~$4,140/mo) at TP=4 FP16
- Or TP=2 INT8 on g5.12xlarge
- [Source: [[sources/aws-ec2-pricing-2026-05]]]

## Successor

**Qwen3-Coder family** (Feb 2026). Qwen3-Coder-Next claims >70% SWE-bench Verified — a massive jump from Qwen2.5-Coder-32B's unreported figure. **For any new deployment focused on tool selection or agentic coding, evaluate Qwen3-Coder first.** Qwen2.5-Coder-32B is still the most heavily-benchmarked open code model.

## Related
- [[models/qwen2.5-coder-14b]] — fallback if 32B fit is too tight
- [[hardware/multi-gpu-options]]
- [[infrastructure/quantization]]
- [[wiki/comparisons/tool-calling-models-on-a10g]]

## Sources
- [[sources/qwen2.5-coder-tech-report]]
- [[sources/code-benchmarks-2026-05]]
- [[sources/vllm-tool-calling-docs]]
