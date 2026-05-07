# Kimi K2 / K2-Instruct / K2-Thinking — Moonshot AI

source_urls:
  - https://huggingface.co/moonshotai/Kimi-K2-Instruct
  - https://huggingface.co/moonshotai/Kimi-K2-Thinking
  - https://moonshotai.github.io/Kimi-K2/
  - https://github.com/MoonshotAI/Kimi-K2
fetched: 2026-05-07

## Kimi-K2-Instruct
- **1T total params / 32B active** (MoE: 384 experts, 8 selected)
- License: **Modified MIT** (open-weight; training data/code not released)
- Context: 128K
- Native format: **Block-FP8** (E4M3) — the released weights are FP8
- Trained on 15.5T tokens

### Benchmarks
- SWE-bench Verified: **65.8%** pass@1 (single attempt, bash + editor tools)
- SWE-bench Verified: 71.6% pass@1 (parallel test-time compute, internal scorer)
- SWE-bench Multilingual: 47.3% pass@1
- K2.5: 76.8% SWE-bench Verified
- K2.6: **80.2%** SWE-bench Verified, 96.4% AIME 2026, 90.5% GPQA-Diamond

## Kimi-K2-Thinking
- Same scale, reasoning-tuned variant
- Always emits `<think>` blocks; vLLM `--reasoning-parser deepseek_r1` (community report)

## Footprint
- FP8 weights: ~1 TB → fits 8× H200 (141 GB × 8 = 1.128 TB) on a single
  **p5e.48xlarge / p5en.48xlarge** with headroom, or 8× B200 on **p6-b200.48xlarge**.
- Will NOT fit 8× H100 80GB at FP8 (640 GB) — needs H200/B200 single-node, or
  multi-node H100.

## vLLM
- `pip install vllm; vllm serve moonshotai/Kimi-K2-Instruct`
- Tool calling: full function-calling spec, autonomous invocation ("agentic")
- Recommended T=0.6, max_tokens=8K for SWE-bench-style eval
- Compatible with SGLang, KTransformers, TensorRT-LLM
