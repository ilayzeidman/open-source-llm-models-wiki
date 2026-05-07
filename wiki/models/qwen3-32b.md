---
tags: [models, generalist, tool-calling, qwen, reasoning]
params: 32.8B
active_params: 32.8B
license: Apache-2.0
context: 32K (131K with YaRN)
release_date: 2025-05-01
last_updated: 2026-05-07
source_count: 1
---

# Qwen3-32B (dense)

Apache-2.0 dense generalist with **hybrid reasoning** — toggles between explicit
`<think>` chain-of-thought and direct response. The dense counterpart to the
Qwen3-Coder MoE line; a strong choice when you want one model that does both
reasoning and tool calling.

## Fit on AWS GPUs

| Box | GPU | Quant | Weights | Max ctx | Verdict |
|---|---|---|---:|---|---|
| g5.xlarge | 1× A10G | AWQ INT4 | ~18 GB | ~7K | ⚠ tight (similar to Qwen2.5-Coder-32B) |
| g6e.xlarge | 1× L40S | AWQ INT4 | ~18 GB | ~80K | ✅ comfortable |
| g6e.xlarge | 1× L40S | FP8 | ~33 GB | ~30K | ✅ fits |
| g5.12xlarge | 4× A10G | FP16 (TP=4) | ~62 GB | ~30K | ✅ full quality |

## Strengths

- Apache-2.0 dense → predictable serving, no MoE expert-routing complexity.
- Hybrid reasoning: `enable_thinking=True` for hard problems, off for cheap turns.
- vLLM mainline support since 0.8.4.
- BFCL v3 family score (Qwen3-235B-A22B reaches 70.8); 32B falls in the high-60s
  on BFCL v3.

## vLLM serving

```
vllm serve Qwen/Qwen3-32B \
  --enable-reasoning --reasoning-parser deepseek_r1 \
  --enable-auto-tool-choice --tool-call-parser hermes \
  --quantization awq_marlin --max-model-len 32768
```

- For thinking mode: T=0.6, top_p=0.95, top_k=20, MinP=0 (no greedy).
- For non-thinking: T=0.7, top_p=0.8.

## Weaknesses

- 32B at AWQ INT4 on A10G (24 GB) leaves only ~7K usable ctx — essentially the
  same fit problem as Qwen2.5-Coder-32B.
- For reasoning + tool calling on A10G, the smaller [[models/qwen3-coder-30b-a3b]]
  MoE (3.3B active) delivers similar tool-calling quality with much faster decode.

## Cost

| Box | $/hr | Note |
|---|---:|---|
| g5.xlarge | $1.006 | tight ctx |
| g6e.xlarge | $1.861 | recommended single-GPU |
| g5.12xlarge | $5.672 | TP=4 FP16 full quality |

## Related

- [[models/qwen3-coder-30b-a3b]] (MoE alternative, faster decode)
- [[models/qwen2.5-coder-32b]] (predecessor)
- [[infrastructure/vllm]] (reasoning + tool-call dual parser)

## Sources

- [[sources/qwen3-dense-cards]]
