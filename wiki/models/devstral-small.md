---
tags: [models, code, agent, mistral]
params: 24B
active_params: 24B
license: Apache-2.0
context: 128K
release_date: 2025-07-01
last_updated: 2026-05-07
source_count: 1
---

# Devstral-Small-2507 (and Devstral-Small-2-2512)

Apache-2.0 dense 24B coder fine-tuned by Mistral × All Hands AI for agentic SWE
scaffolds (OpenHands, SWE-Agent). The clean Apache-2.0 alternative to
[[models/codestral-22b]] (which has the MNPL non-commercial blocker), and the
single best A10G-fitting code model on the wiki by SWE-bench score.

## Fit on AWS GPUs

| Box | GPU | Quant | Weights | Verdict |
|---|---|---|---:|---|
| g5.xlarge | 1× A10G | AWQ INT4 | ~13 GB | ✅✅ comfortable, ~30K ctx |
| g6.xlarge | 1× L4 | AWQ INT4 | ~13 GB | ✅ fits |
| g6e.xlarge | 1× L40S | FP16 (BF16) | ~48 GB | ⚠ JUST fits weights, no KV — use FP8 (~24 GB) instead |
| g6e.xlarge | 1× L40S | FP8 | ~24 GB | ✅ comfortable |

## Strengths

- **SWE-bench Verified 53.6%** (Devstral-Small-2507) → **68.0%** (Devstral-Small-2-2512).
  Best open-source SWE-bench score for a model that fits g5.xlarge.
- Apache-2.0 — fully commercial, zero license risk.
- Dense (no MoE) — predictable batching, mature quant kernels.
- 128K context, Tekken tokenizer.

## Weaknesses

- Dense 24B → smaller batch sizes than Qwen3-Coder-30B-A3B's 3.3B-active MoE.
- Mistral parser path (`--tokenizer_mode mistral --config_format mistral
  --load_format mistral`) is required and a common source of misconfiguration
  for users coming from HuggingFace defaults.

## vLLM serving

```
vllm serve mistralai/Devstral-Small-2507 \
  --tokenizer_mode mistral --config_format mistral --load_format mistral \
  --tool-call-parser mistral --enable-auto-tool-choice \
  --max-model-len 32768
```

For multi-GPU (g5.12xlarge / g6.12xlarge), add `--tensor-parallel-size 2`.

## Variants

- **Devstral-Small-2507**: 24B, SWE-bench Verified 53.6%
- **Devstral-Small-2-24B-Instruct-2512**: 24B, SWE-bench Verified 68.0%
- **Devstral-2-123B-Instruct-2512**: 123B flagship, SWE-bench Verified 72.2%
  — needs g6e.12xlarge (4× L40S = 192 GB, AWQ INT4 ~65 GB) or single H100 80 GB.

## Cost

| Box | $/hr | Variant | Note |
|---|---:|---|---|
| g5.xlarge | $1.006 | Small (24B) AWQ INT4 | original A10G target |
| g6.xlarge | $0.8048 | Small (24B) AWQ INT4 | cheapest |
| g6e.xlarge | $1.861 | Small (24B) FP8 | full-quality single-GPU |
| g6e.12xlarge | $10.49 | 123B AWQ INT4 TP=4 | flagship single-node |

## Related

- [[models/codestral-22b]] (MNPL — superseded by Devstral for commercial use)
- [[models/qwen3-coder-30b-a3b]] (MoE alternative)
- [[infrastructure/vllm]]

## Sources

- [[sources/devstral-small-cards]]

## TODO / verify

- BFCL number for Devstral (Mistral hasn't published a tool-calling-specific score).
- Reproduce SWE-bench Verified on independent harness.
