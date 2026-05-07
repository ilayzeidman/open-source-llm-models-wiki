---
tags: [models, openai, mix-of-experts, reasoning, tool-calling]
params: 117B
active_params: 5.1B
license: Apache-2.0
context: 128K
release_date: 2025-08-05
last_updated: 2026-05-07
source_count: 1
---

# gpt-oss-120b

Apache-2.0 MoE (117B / 5.1B active) — OpenAI's larger open-weight release.
"Near-parity with o4-mini on core reasoning benchmarks; runs on a single 80 GB GPU."

## Fit on AWS GPUs

| Box | GPUs | Mode | Weights | Verdict |
|---|---|---|---:|---|
| p5.48xlarge slice (1× H100) | H100 80GB | MXFP4 native | ~63 GB | ✅✅ fastest, single GPU |
| p6-b200 slice (1× B200) | B200 180GB | MXFP4 / FP4 | ~63 GB | ✅✅✅ best |
| g6e.12xlarge / 24xlarge | 4× L40S | BF16 dequant | ~234 GB total → fits 192 GB? **NO** | ❌ too big BF16 |
| g6e.48xlarge | 8× L40S | AWQ INT4 (community) | ~60 GB | ✅ fits TP=8 with INT4 re-quant |
| p4d.24xlarge | 8× A100 40GB (320 GB) | BF16 dequant | ~234 GB | ✅ fits TP=8 |
| p4de.24xlarge | 8× A100 80GB (640 GB) | BF16 / AWQ | ✅ comfortable |

## Strengths

- "Apache-2.0 frontier reasoning" — Codeforces / MMLU at o3-mini-or-better level.
- Native tool calling, configurable reasoning effort.
- Single H100 fit (MXFP4) → cheapest single-GPU frontier inference if you can
  rent any H100. AWS minimum for that is a p5.48xlarge slice.

## Weaknesses

- AWS does **not sell single-GPU H100/H200** — you pay for 8× even to use 1×
  ($98.32/hr p5.48xlarge). Co-host other models or use Capacity Blocks.
- For pure cost on AWS, gpt-oss-120b is hard to justify vs Qwen3-Coder-30B-A3B
  on g6e.xlarge ($1.86/hr) when SWE-bench scores are within ~5–10 points.

## vLLM serving

```
vllm serve openai/gpt-oss-120b \
  --tool-call-parser openai --enable-auto-tool-choice
```

(Same `vllm==0.10.1+gptoss` build as gpt-oss-20b.)

## Cost

| Box | $/hr | Note |
|---|---:|---|
| p5.48xlarge | $98.32 | single H100 used (7 idle) |
| p4de.24xlarge | $40.97 | 8× A100 80GB BF16 dequant |
| g6e.48xlarge | $30.13 | community AWQ INT4 TP=8 |
| p6-b200.48xlarge | $113.93 | best latency |

## Related

- [[models/gpt-oss-20b]]
- [[models/deepseek-v3.1]], [[models/kimi-k2]] — same multi-node tier
- [[infrastructure/quantization]] (MXFP4 hardware requirement)

## Sources

- [[sources/gpt-oss-cards]]
