---
tags: [models, code, tool-calling, qwen, mix-of-experts]
params: 31B
active_params: 3.3B
license: Apache-2.0
context: 256K
release_date: 2025-08-01
last_updated: 2026-05-07
source_count: 2
---

# Qwen3-Coder-30B-A3B-Instruct

Apache-2.0 MoE coding agent (31B total / 3.3B active). The natural successor to
[[models/qwen2.5-coder-14b]] and [[models/qwen2.5-coder-32b]] — fixes the
flaky-vLLM-`hermes`-parser problem by shipping a dedicated **`qwen3_coder`** parser,
and is the strongest "fits a single A10G/L40S at INT4" code+tools model on the
wiki today.

## Fit on AWS GPUs

| GPU / box | Quant | Weights | Verdict |
|---|---|---:|---|
| 1× A10G (g5.xlarge / g5.2xlarge) | AWQ INT4 | ~17–18 GB | ✅ fits, ~30k usable ctx |
| 1× L4 24GB (g6.xlarge) | AWQ INT4 | ~17–18 GB | ✅ fits, decode-bound on 300 GB/s bw |
| 1× L40S 48GB (g6e.xlarge) | AWQ INT4 | ~17–18 GB | ✅✅ comfortable, full 256K ctx |
| 1× L40S 48GB (g6e.xlarge) | FP8 | ~31 GB | ✅ fits with usable ctx |
| 1× H100 80GB (p5 slice) | BF16 | ~62 GB | ✅ comfortable |

## Strengths

- SWE-bench Verified ~50.3% Pass@1 (Qwen blog) — the highest open-source score for
  any model that fits a single 24 GB GPU.
- BFCL v3 live-simple/multiple ~0.81 (Qwen3-Coder family).
- 256K native context (1M with YaRN) — repo-scale agent loops feasible.
- Apache-2.0 — fully commercial.
- MoE 3.3B active → fast decode, low TPS cost.

## Weaknesses

- MoE expert routing makes some quant kernels (Marlin) less mature than dense; AWQ
  works but throughput lags Qwen2.5-Coder-32B dense at the same VRAM footprint.
- vLLM `qwen3_coder` parser requires vLLM ≥ 0.10.0 — older deployments need upgrade.
- Non-thinking only (no reasoning mode like Qwen3-32B dense). For chain-of-thought,
  pair with [[models/qwen3-32b]] in a dual-model setup.

## vLLM serving notes

```
vllm serve Qwen/Qwen3-Coder-30B-A3B-Instruct \
  --enable-auto-tool-choice --tool-call-parser qwen3_coder \
  --max-model-len 32768 \
  --quantization awq_marlin
```

- Sampling: T=0.7, top_p=0.8, top_k=20, rep_pen=1.05.
- OOM symptom → drop `--max-model-len` to 32768 or lower.

## Cost (AWS, on-demand, us-east-1)

| Box | $/hr | Quant | Note |
|---|---:|---|---|
| g5.xlarge | $1.006 | AWQ INT4 | smallest commercial fit |
| g6.xlarge | $0.8048 | AWQ INT4 | cheapest, slower decode |
| g6e.xlarge | $1.861 | AWQ INT4 / FP8 | recommended sweet spot |
| g5.2xlarge | $1.212 | AWQ INT4 | more vCPU/RAM for batching |

## Related

- [[models/qwen2.5-coder-32b]] (predecessor dense)
- [[models/devstral-small]] (Apache-2.0 dense alternative)
- [[infrastructure/vllm]], [[hardware/aws-gpu-landscape]]
- [[comparisons/models-by-budget]]

## Sources

- [[sources/qwen3-coder-cards]]
- [[sources/qwen2.5-coder-tech-report]]

## TODO / verify

- Reproduce the SWE-bench Verified ~50.3% claim against an independent harness.
- Measure A10G AWQ INT4 throughput; no public number yet.
