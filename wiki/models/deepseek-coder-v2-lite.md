---
tags: [models, code, moe, deepseek]
params: 16B
active_params: 2.4B
license: DeepSeek Model License (commercial use permitted)
context: 128k
release_date: 2024-06-17
last_updated: 2026-05-07
source_count: 3
---

# DeepSeek-Coder-V2-Lite-Instruct

DeepSeek's 16B Mixture-of-Experts code model with **only 2.4B active parameters** per token. Decode is fast because only the active experts run; the full ~16B has to fit in VRAM though.

- HF: `deepseek-ai/DeepSeek-Coder-V2-Lite-Instruct` ([model card](https://huggingface.co/deepseek-ai/DeepSeek-Coder-V2-Lite-Instruct))
- Architecture: DeepSeekMoE; 16B total / 2.4B active

## Fit on A10G (24 GB)
- FP16: ~31 GB → ❌
- INT8: ~16 GB → ⚠ tight, ~6 GB KV
- AWQ INT4: ~10 GB → ✅ comfortable, ~12 GB KV
- **Verdict**: ✅ at AWQ INT4. Fast decode for its capability tier thanks to MoE sparsity.

⚠ Note: No first-party AWQ/GPTQ from `deepseek-ai/`; community AWQs exist but **MoE + AWQ kernel maturity in vLLM should be verified before commitment**. [Source: [[sources/deepseek-coder-v2-paper]]]

## Strengths (DeepSeek-Coder-V2 paper, FP16)

- HumanEval 81.1
- MBPP+ 68.8
- LiveCodeBench (overall) 24.3
- USACO 6.5
- Defects4J 9.2
- FIM mean 86.4
- Aider 44.4
- BBH 61.2; MMLU 60.1
- GSM8K 86.4; MATH 61.8
- ARC-E 88.9 / ARC-C 77.4
- Arena-Hard 38.1

[Source: [[sources/deepseek-coder-v2-paper]], [[sources/code-benchmarks-2026-05]]]

Decode latency profile is closer to a 2.4B dense model than to a 16B dense model — only the active experts run per token. The headline reason to consider it on A10G despite the 16B weight footprint.

## Weaknesses
- **No published BFCL** number. Tool calling is not the model's primary focus.
- **SWE-bench: 0.0** — Lite is sub-threshold for the harness (per the V2 paper).
- Behind Qwen2.5-Coder family on most recent benchmarks.

## License correction

**Correction from the seeded wiki**: the DeepSeek Model License **does permit commercial use** — the V2 series license states it "supports commercial use." Code is under MIT. [Source: [[sources/deepseek-coder-v2-paper]]]

## vLLM serving notes
- vLLM supports DeepSeek-V2 architecture since PR #4650. SGLang also supports with FP8 + Torch Compile.
- Custom DeepSeek role-token chat template (`<｜begin▁of▁sentence｜>` etc.).
- ⚠ **No matching `--tool-call-parser`**: the `deepseek_v3` and `deepseek_v31` parsers are V3-tuned, **not V2-Coder**. For tool-calling, treat as DIY (constrained decoding via xgrammar) until verified. [Source: [[sources/vllm-tool-calling-docs]]]

## Cost (AWS, on-demand, us-east-1)
- Smallest box: **g5.xlarge** ($1.006/hr)
- MoE often gives better throughput per dollar than equivalent-quality dense models. [Source: [[sources/aws-ec2-pricing-2026-05]]]

## Successor / current state

The code-specialist line was folded into general DeepSeek-V3 / V3.1 / R1 — there is **no direct "DeepSeek-Coder-V3-Lite"** at the same 16B/2.4B size. DeepSeek-Coder-V2-Lite remains the canonical small DeepSeek-Coder MoE.

## Related
- [[models/qwen2.5-coder-7b]], [[models/qwen2.5-coder-14b]]
- [[concepts/code-generation]]
- [[infrastructure/quantization]]
- [[wiki/comparisons/tool-calling-models-on-a10g]]

## Sources
- [[sources/deepseek-coder-v2-paper]]
- [[sources/code-benchmarks-2026-05]]
- [[sources/vllm-tool-calling-docs]]
