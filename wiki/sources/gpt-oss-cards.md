---
tags: [source, models, openai, mix-of-experts, fp4]
source_path: raw/gpt-oss-cards.md
source_url: https://openai.com/index/introducing-gpt-oss/
ingested: 2026-05-07
last_updated: 2026-05-07
---

# GPT-OSS — OpenAI's open-weight (gpt-oss-20b / gpt-oss-120b)

OpenAI's first open-weight release in years. Apache-2.0, MoE, MXFP4 native.
**Critical hardware caveat** that reshapes AWS deployment: MXFP4 hardware needs
Hopper or newer (sm_90+).

## Key claims

- **gpt-oss-20b**: 21B total / 3.6B active, Apache-2.0, 128K ctx, MXFP4 native.
  Footprint: ~12–13 GB on disk; runtime 16 GB VRAM minimum.
- **gpt-oss-120b**: 117B total / 5.1B active, Apache-2.0. Runtime ~63 GB → fits
  single 80 GB H100.
- Both use OpenAI's **Harmony response format** (mandatory; vLLM applies automatically).
- Configurable reasoning effort (low/medium/high). gpt-oss-20b SWE-bench Verified:
  37.4 / 53.2 / **60.7** at low/medium/high.
- vLLM: requires custom build `vllm==0.10.1+gptoss`; tool-call parser `openai`.

## Critical hardware caveat — MXFP4 needs sm_90+

- **Hopper (H100/H200) and Blackwell (B200) have MXFP4 tensor cores**.
- **Ampere (A10G, A100), Ada (L4, L40S), Turing (T4) do NOT** — vLLM dequantizes
  MXFP4 → BF16 at load, **doubling VRAM**:
  - gpt-oss-20b BF16-dequantized ≈ 24+ GB → does NOT fit A10G 24 GB at usable ctx;
    fits L40S 48 GB but loses the speed advantage of MXFP4.
  - gpt-oss-120b BF16-dequantized ≈ 234 GB → impossible on G-class without
    re-quantization to AWQ INT4.
- Practical AWS deployment for native MXFP4: **p5.48xlarge** (8× H100) or
  **p6-b200.48xlarge** (8× B200, fastest).

## Pages updated on ingest

- [[models/gpt-oss-20b]] — new
- [[models/gpt-oss-120b]] — new
- [[infrastructure/quantization]] — added MXFP4 row + sm_90+ requirement note
- [[hardware/aws-gpu-landscape]] — MXFP4 compatibility column
- [[comparisons/models-by-budget]] — gpt-oss-20b in p5+/B200 tier (or A10G with
  community AWQ re-quant fallback)
