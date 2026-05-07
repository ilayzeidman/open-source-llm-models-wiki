---
tags: [models, code, generalist, qwen]
params: 7B
active_params: 7B
license: Apache-2.0
context: 128k
release_date: 2024-09
last_updated: 2026-05-07
source_count: 0
---

# Qwen2.5-Coder-7B-Instruct

Alibaba's Qwen team's 7B code model. Strong code reasoning for its size, native 128k context, decent tool calling. Sweet spot for low-cost A10G deployment when 14B-class quality isn't needed.

## Fit on A10G (24 GB)
- FP16/BF16 weights: ~14 GB → ✅ fits with ~9 GB KV-cache budget
- AWQ INT4: ~5 GB → ✅ huge KV-cache budget (~17 GB) → 100k+ ctx feasible at batch=1
- **Verdict**: ✅ comfortable on g5.xlarge

## Strengths
- Code: **HumanEval ~88, MBPP ~83, LiveCodeBench mid-20s** *(unverified — needs source)*
- Tool calling: usable but not specialist-tier *(unverified)*
- Long context: native 128k via YaRN
- Apache-2.0 license

## Weaknesses
- General reasoning weaker than 14B+ models
- Tool calling lags Hermes-3 / xLAM at the same size

## vLLM serving notes
- Tool-call parser: `--tool-call-parser hermes` works for Qwen2.5 *(unverified — verify against latest vLLM)*
- AWQ checkpoints widely available on HF
- YaRN scaling for >32k context may need explicit `--rope-scaling` config

## Cost (AWS, on-demand, us-east-1)
- Smallest box: **g5.xlarge** (~$1.006/hr, ~$734/mo)
- Spot: typically ~$0.30–$0.60/hr
- Throughput on A10G AWQ INT4: ~1500–3000 output tok/s aggregate at batch *(unverified)*

## Related
- [[models/qwen2.5-coder-14b]], [[models/qwen2.5-coder-32b]]
- [[concepts/code-generation]], [[concepts/tool-selection]]
- [[infrastructure/vllm]], [[hardware/a10g-g5xlarge]]

## Sources
- (none yet)

## TODO / verify
- HF model card: Qwen/Qwen2.5-Coder-7B-Instruct
- Qwen2.5-Coder tech report
- BFCL leaderboard entry (if any)
- vLLM tool-parser confirmation for Qwen2.5
