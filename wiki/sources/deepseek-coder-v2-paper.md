---
tags: [source, models, code, deepseek]
source_path: raw/deepseek-coder-v2-paper.md, raw/deepseek-coder-v2-lite-hf-card.md
source_url: https://arxiv.org/abs/2406.11931
ingested: 2026-05-07
last_updated: 2026-05-07
---

# DeepSeek-Coder-V2 paper + Lite-Instruct HF card

DeepSeek's 2024 code MoE paper covering both the 236B/21B-active full model and the 16B/2.4B-active "Lite" variant relevant to this wiki.

## Key claims

### DeepSeek-Coder-V2-Lite-Instruct
- **HF model ID**: `deepseek-ai/DeepSeek-Coder-V2-Lite-Instruct`
- **Architecture**: DeepSeekMoE; 16B total / 2.4B active per token
- **Context**: 128K
- **License**: **DeepSeek Model License** (custom, modified-OpenRAIL-style); code under MIT. **Commercial use is permitted** — the license states the V2 series "supports commercial use." Correction from initial wiki seed (which marked this uncertain).
- **Released**: 2024-06-17

### Benchmarks (from the paper)

| Benchmark | Score |
|---|---|
| HumanEval | 81.1 |
| MBPP+ | 68.8 |
| LiveCodeBench (overall) | 24.3 |
| USACO | 6.5 |
| Defects4J | 9.2 |
| FIM (mean) | 86.4 |
| **SWE-bench** | 0.0 (Lite is sub-threshold for the harness) |
| Aider | 44.4 |
| BBH | 61.2 |
| MMLU | 60.1 |
| GSM8K | 86.4 |
| MATH | 61.8 |
| Arena-Hard | 38.1 |

Not officially reported: BFCL.

### Throughput characteristic
Decode latency profile is closer to a 2.4B dense model than to a 16B dense model — only the active experts run per token. This is the headline reason to consider it on A10G despite the 16B weight footprint.

### vLLM serving
- vLLM support landed via PR #4650
- Custom DeepSeek role-token chat template (`<｜begin▁of▁sentence｜>` etc.)
- ⚠ **No matching `--tool-call-parser`**: the `deepseek_v3` and `deepseek_v31` parsers are V3-tuned, not V2-Coder. Treat tool-calling as DIY (constrained decoding via xgrammar) until verified.
- No first-party AWQ/GPTQ from `deepseek-ai/`; community AWQs exist but MoE + AWQ kernel maturity in vLLM should be verified before commitment.

### Successor status

The code-specialist line was folded into general DeepSeek-V3 / V3.1 / R1 — there is no direct "DeepSeek-Coder-V3-Lite" at the same 16B / 2.4B size. DeepSeek-Coder-V2-Lite remains the canonical small DeepSeek-Coder MoE.

## Citation URLs
- [Paper (arXiv 2406.11931)](https://arxiv.org/abs/2406.11931) / [HTML v1](https://arxiv.org/html/2406.11931v1)
- [HF: deepseek-ai/DeepSeek-Coder-V2-Lite-Instruct](https://huggingface.co/deepseek-ai/DeepSeek-Coder-V2-Lite-Instruct)
- [GitHub: DeepSeek-Coder-V2 repo](https://github.com/deepseek-ai/DeepSeek-Coder-V2)
- [DeepSeek Model License](https://github.com/deepseek-ai/DeepSeek-V2/blob/main/LICENSE-MODEL)

## Pages updated on ingest
- [[models/deepseek-coder-v2-lite]]
- [[wiki/comparisons/tool-calling-models-on-a10g]]
