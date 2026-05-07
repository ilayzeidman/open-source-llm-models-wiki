---
tags: [models, openai, mix-of-experts, reasoning, tool-calling, fp4]
params: 21B
active_params: 3.6B
license: Apache-2.0
context: 128K
release_date: 2025-08-05
last_updated: 2026-05-07
source_count: 1
---

# gpt-oss-20b (OpenAI open-weight)

Apache-2.0 MoE (21B / 3.6B active) released as part of OpenAI's first open-weight
push since GPT-2. Native **MXFP4** weights, **Harmony** response format, and
configurable reasoning effort. Strong tool calling — claimed near-o4-mini parity.

## Fit on AWS GPUs (the MXFP4 caveat)

**MXFP4 hardware requires sm_90+ (H100/H200/B200).** On Ampere/Ada (A10G/L4/L40S),
vLLM dequantizes MXFP4 → BF16 at load, doubling VRAM:

| Box | GPU | Mode | Weights | Verdict |
|---|---|---|---:|---|
| p5 slice (1× H100) | H100 80GB | MXFP4 native | ~13 GB | ✅✅ fastest |
| p6-b200 slice | B200 180GB | MXFP4 / FP4 | ~13 GB | ✅✅✅ best on AWS |
| g6e.xlarge | L40S 48GB | BF16 dequant | ~24 GB | ✅ fits, loses MXFP4 speed gain |
| g5.xlarge | A10G 24GB | BF16 dequant | ~24 GB | ❌ no headroom for KV — community AWQ INT4 quant fits (~7 GB) but slower |
| g4dn.xlarge | T4 16GB | — | — | ❌ doesn't fit MXFP4 dequant |

## Strengths

- **SWE-bench Verified 60.7% at high reasoning** — strongest small open-source agent.
- Configurable reasoning effort (low/medium/high) — cost/quality knob per request.
- Native tool calling, structured outputs, Python execution.
- Apache-2.0 + 128K ctx + tiny on-disk size (~13 GB).

## Weaknesses

- **AWS practical floor is g6e.xlarge ($1.86/hr), not g5.xlarge** — the original
  research target is suboptimal for this model unless you accept community-quantized
  AWQ INT4 with reduced quality.
- Harmony format mandatory — clients written for ChatML need a wrapper.
- vLLM requires custom build: `vllm==0.10.1+gptoss`.

## vLLM serving

```
uv pip install --pre vllm==0.10.1+gptoss \
  --extra-index-url https://wheels.vllm.ai/gpt-oss/

vllm serve openai/gpt-oss-20b \
  --tool-call-parser openai --enable-auto-tool-choice
```

## Cost

| Box | $/hr | Quant | Note |
|---|---:|---|---|
| g6e.xlarge | $1.861 | BF16 (dequant) | recommended single-GPU AWS |
| p5.48xlarge slice | $98.32 / 8 ≈ $12.29 | MXFP4 native | only if H100 needed |
| g5.xlarge | $1.006 | community AWQ INT4 | fits but quality cost |

## Related

- [[models/gpt-oss-120b]]
- [[infrastructure/quantization]] (MXFP4 + sm_90+ requirement)
- [[hardware/aws-gpu-landscape]]

## Sources

- [[sources/gpt-oss-cards]]

## TODO / verify

- Independent benchmarks of community AWQ INT4 gpt-oss-20b on A10G — does the quant
  preserve the 60.7% SWE-bench number?
