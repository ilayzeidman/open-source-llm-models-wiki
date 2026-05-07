# Qwen3-Coder Family — HF cards + Qwen blog

source_urls:
  - https://huggingface.co/Qwen/Qwen3-Coder-30B-A3B-Instruct
  - https://huggingface.co/Qwen/Qwen3-Coder-480B-A35B-Instruct
  - https://qwenlm.github.io/blog/qwen3-coder/
  - https://docs.vllm.ai/projects/recipes/en/latest/Qwen/Qwen3-Coder-480B-A35B.html
fetched: 2026-05-07

## Qwen3-Coder-30B-A3B-Instruct
- 31B total params, **3.3B active** (MoE: 128 experts, 8 activated, 48 layers)
- License: **Apache-2.0**
- Context: 256K native, 1M with YaRN
- Tensor type: BF16
- Recommended sampling: T=0.7, top_p=0.8, top_k=20, rep_pen=1.05
- Non-thinking mode only (no `<think>` blocks); replaces Qwen2.5-Coder for agentic uses
- vLLM recipes confirm tool-call parser **`qwen3_coder`** (vLLM ≥ 0.10.0)
- Quantized variants: 136+ on HF (AWQ, GPTQ, GGUF Q4_K_M)
- Footprint: ~62 GB BF16 weights → fits a single 80 GB H100 / 1× B200; AWQ INT4
  ≈ 17–18 GB → **fits A10G / L4 / L40S single GPU**

## Qwen3-Coder-480B-A35B-Instruct
- 480B total params, **35B active** (MoE)
- License: Apache-2.0
- Context: 256K native, 1M with YaRN
- SWE-bench Verified: **66.5% Pass@1** (Qwen blog, 250-turn agent)
- BFCL v3: tool-calling competitive with proprietary frontier
- vLLM serving: `--enable-expert-parallel --tensor-parallel-size 8 --tool-call-parser qwen3_coder`
- Footprint: ≈ 960 GB BF16 → needs 8× H100/H200/B200 (or 16× with FP16 across two p5)
- Production-grade open-source coding agent flagship

## Qwen3-Coder-30B-A3B-Instruct benchmarks (Qwen blog)
- SWE-bench Verified: ~50.3% Pass@1
- LiveCodeBench: strong multi-turn agent performance
- BFCL v3 (live simple): 0.8171 avg accuracy
- Tool-calling format: native; vLLM `qwen3_coder` parser

## vLLM tool-call parser
- Mainline `qwen3_coder` parser added in vLLM 0.10.0 (early 2025-Q4)
- Stable in 0.20.x
- Replaces the flaky `hermes` parser path that mis-handled Qwen2.5-Coder JSON fences
