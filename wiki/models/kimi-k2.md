---
tags: [models, moonshot, mix-of-experts, agent]
params: 1T
active_params: 32B
license: Modified MIT
context: 128K
release_date: 2025-07-01
last_updated: 2026-05-07
source_count: 1
---

# Kimi-K2-Instruct (and K2-Thinking, K2.5, K2.6)

Moonshot AI's 1T-parameter MoE — the largest open-weight model in the wiki by
total params. Modified MIT, Block-FP8 native. Versioned: K2 → K2.5 → K2.6 with
SWE-bench Verified climbing 65.8 → 76.8 → **80.2%**.

## Fit on AWS GPUs

| Box | GPUs (total VRAM) | Mode | Verdict |
|---|---|---|---|
| p5.48xlarge | 8× H100 80GB (640 GB) | FP8 (~1 TB) | ❌ does not fit FP8 |
| p5e.48xlarge | 8× H200 141GB (1.128 TB) | FP8 native | ✅ comfortable |
| p5en.48xlarge | 8× H200 141GB | FP8 native | ✅ comfortable |
| p6-b200.48xlarge | 8× B200 180GB (1.44 TB) | FP8 / FP4 | ✅✅ best |
| Multi-node (2× p4d / p5) | 16× GPU | FP8 / BF16 | ✅ if you can do multi-node |

**Single-node H100 cannot run Kimi-K2 at FP8** — needs H200 or B200 for single-node,
or multi-node H100. This is the single biggest dividing line in the AWS lineup.

## Strengths

- **SWE-bench Verified: 65.8% (K2-Instruct), 76.8% (K2.5), 80.2% (K2.6)**
  — K2.6 is the highest open-source SWE-bench Verified number on the wiki.
- Modified MIT — open weights, commercially usable.
- Block-FP8 native (no quantization step).
- 1T params / 32B active gives exceptional pretraining-quality + fast decode.

## Weaknesses

- AWS deployment requires **Capacity Blocks for ML** for p5e/p5en (H200) — not
  on-demand.
- Cost: $40+/hr realistic.
- Training data not released — "open-weight", not "open-source" in the strict sense.

## vLLM serving

```
vllm serve moonshotai/Kimi-K2-Instruct \
  --tensor-parallel-size 8 \
  --enable-auto-tool-choice
```

Tool calling: native function-calling spec, autonomous invocation. T=0.6.

## Cost

| Box | $/hr | Note |
|---|---:|---|
| p5e.48xlarge (Capacity Block) | ~$45 reserve fee | recommended H200 path |
| p6-b200.48xlarge | $113.93 | best on-demand option |

## Related

- [[models/deepseek-v3.1]], [[models/qwen3-coder-480b]] — same tier; K2.6 leads on SWE-bench Verified
- [[hardware/aws-gpu-landscape]]

## Sources

- [[sources/kimi-k2-cards]]
