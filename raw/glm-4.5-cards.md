# GLM-4.5 / GLM-4.5-Air family — HF cards + paper

source_urls:
  - https://huggingface.co/zai-org/GLM-4.5
  - https://huggingface.co/zai-org/GLM-4.5-Air
  - https://github.com/zai-org/GLM-4.5
  - arXiv: 2508.06471 (GLM-4.5 ARC paper)
  - https://docs.z.ai/guides/llm/glm-4.6
fetched: 2026-05-07

## GLM-4.5 ("ARC: Agentic, Reasoning, Coding")
- 355B total / **32B active** (MoE)
- License: MIT
- Hybrid reasoning (thinking / direct response)
- FP8 quantized variants released alongside BF16

### Benchmarks
- SWE-bench Verified: **64.2%**
- TAU-Bench: 70.1%
- AIME 2024: 91.0%
- 3rd among proprietary + open-source on aggregate (paper)

## GLM-4.5-Air
- **106B total / 12B active** (MoE)
- License: MIT
- Hybrid reasoning
- Aggregate score 59.8 across 12 benchmarks (vs 63.2 for GLM-4.5)
- Footprint: ~212 GB BF16 → multi-GPU; FP8 ~106 GB → single H100/L40S 8×;
  AWQ INT4 ≈ 53 GB → fits g6e.48xlarge (8× L40S = 384 GB) easily, or 2× L40S
  with TP=2 (96 GB) — borderline.

## GLM-4.6 (notes from Z.AI docs)
- Iterative improvement over 4.5; same scale family
- Competitive with DeepSeek-V3.1-Terminus and Claude Sonnet 4 (vendor framing)

## vLLM tool-call parser
- vLLM mainline supports GLM-4 / GLM-4.5 via `--tool-call-parser glm45`
  (introduced in 0.10.x). Implementation: `vllm/model_executor/models/glm4_moe_mtp.py`.

## AWS fit
- GLM-4.5-Air at FP8: fits 2× L40S (96 GB) — **g6e.12xlarge ($10.49/hr)** or
  g6e.48xlarge with headroom.
- GLM-4.5 (full 355B): needs p5.48xlarge / p4d.24xlarge with TP=8.
