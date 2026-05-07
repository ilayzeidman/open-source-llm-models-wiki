---
tags: [models, code, generalist, qwen]
params: 14B
active_params: 14B
license: Apache-2.0
context: 128k
release_date: 2024-11
last_updated: 2026-05-07
source_count: 0
---

# Qwen2.5-Coder-14B-Instruct

Alibaba's 14B code model. **The current sweet spot** for A10G — much stronger than 7B on real coding tasks while still leaving comfortable KV-cache budget at AWQ INT4.

## Fit on A10G (24 GB)
- FP16/BF16 weights: ~28 GB → ❌ won't fit
- AWQ INT4: ~8–9 GB → ✅ comfortable; ~14 GB KV-cache budget → ~50k ctx at batch=1
- INT8: ~14 GB → ⚠ tight; ~8 GB for KV cache
- **Verdict**: ✅ at AWQ INT4 — recommended default for code-focused workloads on g5.xlarge

## Strengths
- Code: **HumanEval ~89, MBPP ~84, LiveCodeBench ~33** *(unverified — needs source)*
- Repo-scale coding stronger than 7B sibling
- 128k context, Apache-2.0
- Among the best open-source code models that fit on a single A10G

## Weaknesses
- Tool-calling not as strong as specialist models like [[models/xlam-7b]] or [[models/hermes-3-llama-3.1-8b]] *(unverified)*
- General-knowledge tasks behind same-size generalists like [[models/phi-3-medium-14b]]

## vLLM serving notes
- Tool-call parser: `--tool-call-parser hermes` *(unverified — verify against latest vLLM)*
- Pre-quantized AWQ checkpoints on HF (Qwen team publishes official AWQ)
- Pair with `--enable-prefix-caching` for tool-heavy workloads

## Cost (AWS, on-demand, us-east-1)
- Smallest box: **g5.xlarge** (~$1.006/hr, ~$734/mo)
- Spot: typically ~$0.30–$0.60/hr

## Related
- [[models/qwen2.5-coder-7b]], [[models/qwen2.5-coder-32b]]
- [[concepts/code-generation]], [[concepts/tool-selection]]
- [[infrastructure/vllm]], [[infrastructure/quantization]]
- [[comparisons/tool-calling-models-on-a10g]]

## Sources
- (none yet)

## TODO / verify
- HF model card: Qwen/Qwen2.5-Coder-14B-Instruct
- Qwen2.5-Coder tech report
- BFCL leaderboard entry
- Aider leaderboard entry
- LiveCodeBench score
