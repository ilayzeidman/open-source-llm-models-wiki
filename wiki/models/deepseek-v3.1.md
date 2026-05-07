---
tags: [models, deepseek, mix-of-experts, reasoning, agent]
params: 671B
active_params: 37B
license: MIT
context: 128K
release_date: 2025-08-21
last_updated: 2026-05-07
source_count: 1
---

# DeepSeek-V3.1

671B-total / 37B-active MoE, MIT license, native FP8 (UE8M0). The first
open-source model to clear 60% on SWE-bench Verified agent-mode.

## Fit on AWS GPUs

| Box | GPUs | Mode | Verdict |
|---|---|---|---|
| p5.48xlarge | 8× H100 80GB (640 GB) | FP8 (~671 GB weights) | ❌ FP8 weights exceed 640 GB; need INT4 quant or larger box |
| p4de.24xlarge | 8× A100 80GB (640 GB) | AWQ INT4 (~340 GB) | ✅ NVLink, room for KV |
| p5e.48xlarge | 8× H200 141GB (1.13 TB) | FP8 / BF16 | ✅✅ smallest single-node FP8 fit |
| p5en.48xlarge | 8× H200 141GB | FP8 / BF16 | ✅✅ comfortable |
| p6-b200.48xlarge | 8× B200 180GB (1.44 TB) | FP8 / FP4 | ✅✅✅ best latency |
| g6e.48xlarge | 8× L40S 48GB (384 GB) | AWQ INT4 (~340 GB) | ⚠ borderline; PCIe TP=8 |

**FP8 native single-node fit requires p5e/p5en (H200) or p6-b200.** p5.48xlarge
(H100) is too small for FP8; use AWQ INT4 on p4de or g6e.48xlarge if cost-sensitive.
Cannot run on any G-class smaller than g6e.48xlarge.

## Strengths

- **SWE-bench Verified Agent: 66.0** (non-thinking) — among open-source frontier.
- LiveCodeBench 56.4 / 74.8 (think); Aider-Polyglot 68.4 / 76.3 (think).
- MIT license — strongest open-source reasoning model with permissive license.
- Hybrid reasoning (think / non-think toggle).
- Native FP8 weights — no separate quantization step.

## vLLM serving

```
vllm serve deepseek-ai/DeepSeek-V3.1 \
  --tensor-parallel-size 8 \
  --enable-auto-tool-choice --tool-call-parser deepseek_v3 \
  --enable-reasoning --reasoning-parser deepseek_r1 \
  --max-model-len 32768
```

## Cost (AWS, on-demand)

| Box | $/hr | $/month (24×7) | Notes |
|---|---:|---:|---|
| p4de.24xlarge | $40.97 | ~$29,900 | AWQ INT4 TP=8, NVLink |
| p5.48xlarge | $98.32 | ~$71,800 | INT4 only (FP8 doesn't fit) |
| p5e/p5en.48xlarge | Capacity Blocks (~$39.8–$45.8 reserve) | ~$29k–$33k+ reserve | recommended FP8 |
| p6-b200.48xlarge | $113.93 | ~$83,200 | fastest |

## Related

- [[models/kimi-k2]], [[models/qwen3-coder-480b]], [[models/glm-4.5-air]]
  — same multi-node tier
- [[hardware/aws-gpu-landscape]]

## Sources

- [[sources/deepseek-v3-r1-family]]
