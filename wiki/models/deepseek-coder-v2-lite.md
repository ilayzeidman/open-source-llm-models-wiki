---
tags: [models, code, moe, deepseek]
params: 16B
active_params: 2.4B
license: DeepSeek License (permissive, commercial OK)
context: 128k
release_date: 2024-06
last_updated: 2026-05-07
source_count: 0
---

# DeepSeek-Coder-V2-Lite-Instruct

DeepSeek's 16B Mixture-of-Experts code model with **only 2.4B active parameters** per token. Decode is fast because only the active experts run; the full ~16B has to fit in VRAM though.

## Fit on A10G (24 GB)
- FP16: ~31 GB → ❌
- INT8: ~16 GB → ⚠ tight, ~6 GB KV
- AWQ INT4: ~10 GB → ✅ comfortable, ~12 GB KV
- **Verdict**: ✅ at AWQ INT4. Fast decode for its capability tier thanks to MoE sparsity.

## Strengths
- Code: **HumanEval ~81, MBPP ~75, LiveCodeBench mid-20s** *(unverified — needs source)*
- Strong on multilingual code (Python, Java, C++, etc.)
- Decode latency closer to a 2.4B dense model than to a 16B dense model
- 128k context

## Weaknesses
- Tool calling: not a primary focus; may need constrained decoding for reliable JSON *(unverified)*
- MoE routing overhead — not all kernels are equally optimized; verify vLLM throughput
- Behind Qwen2.5-Coder family on the most recent benchmarks *(unverified)*

## vLLM serving notes
- vLLM has supported DeepSeek-V2 architecture since mid-2024 *(unverified — verify version)*
- Tool-call parser: no dedicated parser as of writing; rely on constrained decoding for tool-call format compliance
- AWQ checkpoints available on HF

## Cost (AWS, on-demand, us-east-1)
- Smallest box: **g5.xlarge** (~$1.006/hr)
- MoE often gives better throughput per dollar than equivalent-quality dense models

## Related
- [[models/qwen2.5-coder-7b]], [[models/qwen2.5-coder-14b]]
- [[concepts/code-generation]]
- [[infrastructure/quantization]]

## Sources
- (none yet)

## TODO / verify
- HF model card: deepseek-ai/DeepSeek-Coder-V2-Lite-Instruct
- DeepSeek-Coder-V2 paper
- vLLM throughput data for MoE on A10G
- BFCL score (if available)
