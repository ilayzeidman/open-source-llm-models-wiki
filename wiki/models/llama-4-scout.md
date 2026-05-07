---
tags: [models, llama, mix-of-experts, multimodal]
params: 109B
active_params: 17B
license: Llama 4 Community
context: 10M
release_date: 2025-04-05
last_updated: 2026-05-07
source_count: 1
---

# Llama-4-Scout-17B-16E-Instruct

Meta's MoE (109B / 17B active, 16 experts), natively multimodal, **10M-token
context**. Smallest single-box-deployable Llama 4 variant.

## Fit on AWS GPUs

| Box | Mode | Weights | Verdict |
|---|---|---:|---|
| 1× H100 80GB (p5 slice) | INT4 on-the-fly | ~55 GB | ✅ Meta's reference fit |
| g6e.xlarge | INT4 | ~55 GB | ❌ exceeds 48 GB |
| g6e.12xlarge | FP8 TP=2 | ~109 GB | ❌ exceeds 96 GB cap (2× L40S) |
| g6e.12xlarge | **FP8 TP=4** | ~28 GB/GPU | ✅ smallest single-node FP8 fit (4× L40S = 192 GB) |
| g6e.48xlarge | INT4 TP=8 | ~7 GB/GPU | ✅ comfortable |
| p4d.24xlarge | BF16 TP=8 | ~218 GB | ✅ NVLink, full quality |

## Strengths

- 10M-token context (Meta claim) — repo-scale and document-scale agents feasible.
- Multimodal natively (text + image input).
- 17B active → fast decode despite 109B total.

## Weaknesses

- License: **Llama 4 Community License** — non-OSI; commercial use OK only for
  organizations under 700M MAU. Not Apache/MIT.
- Coding: LiveCodeBench 32.8 (vs Qwen3-Coder-30B-A3B ~50%, Devstral-Small-2 68% SWE-V).
- Tool-calling reliability: community reports inconsistent JSON adherence.

## vLLM serving

```
vllm serve meta-llama/Llama-4-Scout-17B-16E-Instruct \
  --tensor-parallel-size 2 \
  --enable-auto-tool-choice --tool-call-parser llama3_json
```

(`llama3_json` parser path is the Llama 4 default per current vLLM mainline.)

## Cost

| Box | $/hr |
|---|---:|
| g6e.48xlarge | $30.13 |
| p4d.24xlarge | $32.77 |
| p5.48xlarge slice | $98.32 |

## Related

- [[models/qwen3-coder-30b-a3b]], [[models/glm-4.5-air]] (Apache/MIT alternatives)
- [[models/llama-3.3-70b-instruct]] (predecessor)

## Sources

- [[sources/llama-4-cards]]

## TODO / verify

- Llama 4 BFCL v3 score (Meta did not publish; community evals pending).
- Real-world tool-calling reliability vs Qwen3 / GLM-4.5.
