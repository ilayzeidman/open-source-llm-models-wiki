---
tags: [models, code, tool-calling, qwen, mix-of-experts, multi-node]
params: 480B
active_params: 35B
license: Apache-2.0
context: 256K
release_date: 2025-07-01
last_updated: 2026-05-07
source_count: 1
---

# Qwen3-Coder-480B-A35B-Instruct

The flagship open-source agentic-coding model from Qwen. 480B total / 35B active,
Apache-2.0, **66.5% Pass@1 on SWE-bench Verified** (Qwen blog).

## Fit on AWS GPUs

| Box | GPUs | VRAM | Quant | Verdict |
|---|---|---:|---|---|
| p5.48xlarge | 8× H100 80GB | 640 GB | FP8 | ⚠ tight (~480 GB FP8 weights + KV) |
| p5e.48xlarge | 8× H200 141GB | 1,128 GB | FP8 / BF16 | ✅ comfortable |
| p6-b200.48xlarge | 8× B200 180GB | 1,440 GB | BF16 / FP8 / FP4 | ✅✅ best latency |
| g6e.48xlarge | 8× L40S 48GB | 384 GB | AWQ INT4 (~120 GB) | ✅ fits but PCIe-only TP=8 — high comms overhead |

## Strengths

- **SWE-bench Verified 66.5%** — among open-source frontier (DeepSeek-V3.1 66.0,
  Kimi-K2 65.8, GLM-4.5 64.2).
- Apache-2.0 — only frontier-tier open-source coding model with permissive license
  (vs Llama 4 community license).
- BFCL v3 strong; 256K ctx.

## Weaknesses

- 8× GPU minimum; not deployable on any G-class without TP=8 PCIe (slow).
- Hourly cost: $30–$114/hr depending on box.
- New model; ecosystem (long-running agent harness integrations) still maturing.

## vLLM serving

```
vllm serve Qwen/Qwen3-Coder-480B-A35B-Instruct \
  --tensor-parallel-size 8 \
  --enable-expert-parallel \
  --enable-auto-tool-choice --tool-call-parser qwen3_coder \
  --max-model-len 32000
```

(Per official vLLM recipe.)

## Cost (AWS, on-demand)

| Box | $/hr | $/month (24×7) |
|---|---:|---:|
| g6e.48xlarge | $30.13 | ~$22,000 |
| p4d.24xlarge | $32.77 | ~$23,900 |
| p5.48xlarge | $98.32 | ~$71,800 |
| p6-b200.48xlarge | $113.93 | ~$83,200 |

## Related

- [[models/qwen3-coder-30b-a3b]] (smaller sibling, A10G fit)
- [[models/deepseek-v3.1]], [[models/kimi-k2]], [[models/glm-4.5-air]]
- [[hardware/aws-gpu-landscape]], [[comparisons/models-by-budget]]

## Sources

- [[sources/qwen3-coder-cards]]
