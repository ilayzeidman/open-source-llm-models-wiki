---
tags: [comparison, master-table]
last_updated: 2026-05-07
source_count: 17
---

# Tool-Calling & Code Models on A10G — Master Comparison

The artifact that answers the original (A10G-locked) research question from
[[wiki/overview]]: which open-source LLMs are credible for tool selection /
complex code on a single A10G (24 GB) via [[infrastructure/vllm]] under
[[infrastructure/nvidia-dynamo]]?

> **Companion table (2026-05 expansion)**: see [[comparisons/models-by-budget]]
> for the same question across the full AWS GPU spectrum (g4dn → p6-b200), and
> [[hardware/aws-gpu-landscape]] for the GPU/instance menu.

All numbers below are from official primary sources (HF model cards, tech reports, vendor blogs) cross-checked May 2026, and are **FP16/BF16 unless explicitly marked otherwise**. No public BFCL/SWE-bench/LiveCodeBench/EvalPlus runs exist at AWQ INT4 — the only quantized public scores in this domain are Aider's Ollama Q8 entries.

## How to read

- **VRAM (rec quant)** = expected weight footprint at the recommended quant for A10G fit
- **Max ctx (24 GB, batch=1)** = practical context length with the rest of VRAM as KV cache
- **BFCL** = score and version (V1/V2/V3); see [[concepts/benchmarks]]
- **HE / LCB** = HumanEval (FP16) / LiveCodeBench Pass@1 with date slice
- **vLLM parser** = `--tool-call-parser` flag value (or "no mainline" if not supported)
- **License** = abbreviated; **NC** = non-commercial
- **Verdict** = ✅ great fit / ⚠ compromise / ❌ doesn't fit single A10G / 🚫 license blocker

## Single-A10G candidates (sorted by tool-calling utility for commercial deployment)

| Model | Params | Quant | VRAM | Max ctx | BFCL | HumanEval | LCB | License | vLLM parser | $/hr | Verdict |
|---|---|---|---|---|---|---|---|---|---|---:|---|
| [[models/qwen3-coder-30b-a3b\|Qwen3-Coder-30B-A3B-Instruct]] | 31B/3.3B MoE | AWQ INT4 | ~17–18 GB | ~30k | family BFCL v3 ~0.81 live | not reported | SWE-V ~50.3 | Apache-2.0 | `qwen3_coder` | $1.006 | ✅ **NEW code+tools default** |
| [[models/devstral-small\|Devstral-Small-2507]] | 24B | AWQ INT4 | ~13 GB | ~30k | not published | not reported | SWE-V **53.6** (v2-2512: 68.0) | Apache-2.0 | `mistral` | $1.006 | ✅ **best A10G SWE-bench, Apache** |
| [[models/qwen3-32b\|Qwen3-32B (dense, hybrid reasoning)]] | 32B | AWQ INT4 | ~18 GB | ~7k ⚠ | family BFCL v3 ~70.8 (235B) | not reported | not reported | Apache-2.0 | `hermes` + `deepseek_r1` | $1.006 | ⚠ tight ctx (consider g6e) |
| [[models/granite-3.1-8b\|Granite-3.3-8B-Instruct]] | 8B | AWQ INT4 | ~5 GB | 100k+ | none on card; "best-in-class" claim | **89.73** | not reported | Apache-2.0 | `granite` | $1.006 | ✅ **commercial default for tool calling** |
| [[models/hermes-3-llama-3.1-8b\|Hermes-3-Llama-3.1-8B]] | 8B | AWQ INT4 | ~5 GB | 100k+ | "~91% well-formed JSON" — not BFCL | not on card | — | Llama 3.1 community | `hermes` | $1.006 | ✅ tool-calling specialist |
| [[models/llama-3.1-8b-instruct\|Llama-3.1-8B-Instruct]] | 8B | AWQ INT4 | ~5 GB | 100k+ | **76.1 (V2)** | 72.6 | — | Llama 3.1 community | `llama3_json` | $1.006 | ✅ generalist baseline |
| [[models/qwen2.5-coder-7b\|Qwen2.5-Coder-7B-Instruct]] | 7B | AWQ INT4 | ~4 GB | 100k+ | not on leaderboard | **88.4** | 18.2 (07–11/2024) | Apache-2.0 | `hermes` ⚠ flaky | $1.006 | ✅ cheap code 7B |
| [[models/qwen2.5-coder-14b\|Qwen2.5-Coder-14B-Instruct]] | 14B | AWQ INT4 | ~9 GB | ~50k | not on leaderboard | **89.6** | 23.4 | Apache-2.0 | `hermes` ⚠ flaky | $1.006 | ✅ **code+tools sweet spot** |
| [[models/mistral-small-24b\|Mistral-Small-3.2-24B-Instruct-2506]] | 24B | AWQ INT4 | ~13–14 GB | ~30k | not published | 88.4 (3.1) / 92.90 HE+ (3.2) | not reported | Apache-2.0 | `mistral` | $1.006 | ✅ generalist 24B |
| [[models/deepseek-coder-v2-lite\|DeepSeek-Coder-V2-Lite-Instruct]] | 16B MoE / 2.4B active | AWQ INT4 | ~10 GB | ~40k | not on leaderboard | 81.1 | 24.3 (Q3/2024) | DeepSeek (commercial OK) | none — V3 parsers not V2-tuned | $1.006 | ✅ fast decode (MoE) |
| [[models/qwen2.5-coder-32b\|Qwen2.5-Coder-32B-Instruct]] | 32B | AWQ INT4 | ~19 GB | **~7k ⚠** | not on leaderboard | **92.7** | **31.4** (07–11/2024) | Apache-2.0 | `hermes` ⚠ flaky | $1.006 | ⚠ tight ctx — see g5.12xl |
| [[models/phi-3-medium-14b\|Phi-3-Medium-128k]] | 14B | AWQ INT4 | ~9 GB | ~50k | not on leaderboard | 58.5 | not reported | MIT | **no mainline parser** | $1.006 | 🚫 no tool calling |
| [[models/codestral-22b\|Codestral-22B-v0.1]] | 22B | AWQ INT4 | ~13 GB | ~30k | not published | 86.6 | community only | **MNPL — non-commercial** | `mistral` (not Codestral-trained) | $1.006 | 🚫 license + no tool training |
| [[models/functionary-small\|functionary-small-v3.2]] | 8B | AWQ INT4 | ~5 GB | 100k+ | 82.82 (v2-era) | not on card | — | MIT (+ Llama 3.1 community) | **no mainline parser** | $1.006 | ⚠ needs MeetKai fork |
| [[models/xlam-7b\|Llama-xLAM-2-8b-fc-r]] | 8B | AWQ INT4 | ~5 GB | 100k+ | "SOTA on V3" qualitative | — | — | **CC-BY-NC-4.0 — research only** | `xlam` | $1.006 | 🚫 NC license |
| [[models/xlam-7b\|xLAM-7b-fc-r]] (v1) | 7B | AWQ INT4 | ~4 GB | 4K | 88.24 (V1, 07/2024) | — | — | **CC-BY-NC-4.0** | plugin (`xlam_tool_call_parser.py`) | $1.006 | 🚫 NC + 4K ctx |

## Multi-GPU and bigger boxes (out of A10G scope; see [[comparisons/models-by-budget]])

| Model | Params | Best fit | Box | $/hr | Notes |
|---|---|---|---|---:|---|
| [[models/qwen2.5-coder-32b\|Qwen2.5-Coder-32B]] | 32B | AWQ INT4 80K ctx | **g6e.xlarge** | **$1.861** | **single-GPU 48 GB beats g5.12xl** |
| [[models/qwen3-coder-30b-a3b\|Qwen3-Coder-30B-A3B]] | 31B MoE | FP8 256K ctx | g6e.xlarge | $1.861 | full quality single-GPU |
| [[models/qwen2.5-coder-32b\|Qwen2.5-Coder-32B]] | 32B | TP=2 INT8 or TP=4 FP16 | g5.12xlarge | $5.672 | full-quality 32B coder |
| [[models/glm-4.5-air]] | 106B/12B MoE | FP8 TP=4 | g6e.12xlarge | $10.493 | MIT; parent GLM-4.5 SWE-V 64.2 / TAU 70.1 (Air-specific unpublished) |
| [[models/llama-3.3-70b-instruct\|Llama-3.3-70B-Instruct]] | 70B | TP=8 AWQ INT4 | g5.48xlarge | $16.288 | cheapest 70B fit (PCIe) |
| [[models/llama-3.3-70b-instruct\|Llama-3.3-70B-Instruct]] | 70B | TP=8 FP16 (NVLink) | p4d.24xlarge | $32.7726 | best Ampere/NVLink latency |
| Hermes-4-70B | 70B | TP=8 AWQ INT4 | g5.48xlarge | $16.288 | Hermes specialist scaled up |
| [[models/llama-4-scout]] | 109B/17B MoE | INT4 TP=8 | g6e.48xlarge | $30.13 | 10M ctx, Llama 4 community |
| [[models/qwen3-coder-480b]] | 480B/35B MoE | FP8 TP=8 | p5e/p5en | ~$45+ | Apache-2.0, SWE-V 66.5 |
| [[models/deepseek-v3.1]] | 671B/37B MoE | FP8 TP=8 | p5e/p5en | ~$45+ | MIT, SWE-V 66.0 |
| [[models/kimi-k2]] | 1T/32B MoE | FP8 TP=8 | p5e/p5en or p6-b200 | $45+/$113.93 | K2.6 SWE-V **80.2** |
| [[models/gpt-oss-120b]] | 117B/5.1B MoE | MXFP4 native | p5.48xl slice / p6-b200 | $98.32+ | requires sm_90+ for MXFP4 native |

See [[hardware/multi-gpu-options]] for the full cost menu.

## Recommended starting points by use case (updated 2026-05)

- **Just tool calling, commercial use, cheapest** → [[models/granite-3.1-8b|Granite-3.3-8B-Instruct]] (Apache-2.0, `granite` parser, HE 89.73).
- **Code + tool calling, commercial use** → [[models/qwen3-coder-30b-a3b|Qwen3-Coder-30B-A3B-Instruct]] AWQ INT4 (Apache-2.0, mainline `qwen3_coder` parser). **Replaces** the older Qwen2.5-Coder-14B recommendation — same A10G fit, better SWE-bench, and the new parser fixes the flaky-`hermes` issue.
- **Best agentic SWE-bench on A10G** → [[models/devstral-small|Devstral-Small-2507]] / Devstral-Small-2-2512 (Apache-2.0, `mistral` parser, SWE-V 53.6 → 68.0). Replaces Codestral-22B (MNPL non-commercial blocker).
- **Best tool-calling specialist, commercial use** → [[models/hermes-3-llama-3.1-8b|Hermes-3-Llama-3.1-8B]] (Llama community, mainline `hermes` parser).
- **Maximum code quality on a single GPU** → step up to **g6e.xlarge ($1.861/hr, L40S 48 GB)**: run Qwen2.5-Coder-32B / [[models/qwen3-32b]] / Qwen3-Coder-30B-A3B at FP8 with full ctx, no multi-GPU comms tax. See [[hardware/g6e-l40s]].
- **70B generalist** → [[models/llama-3.3-70b-instruct|Llama-3.3-70B-Instruct]] AWQ INT4 TP=4 on **g6e.12xlarge ($10.493/hr)** — cheaper than g5.48xlarge and faster.
- **Frontier open-source code+tools** → [[models/qwen3-coder-480b]] (Apache-2.0) / [[models/deepseek-v3.1]] (MIT) / [[models/kimi-k2]] (mod-MIT) on p5e/p5en (H200) or p6-b200.
- **Apache-2.0 frontier reasoning** → [[models/gpt-oss-20b]] / [[models/gpt-oss-120b]] (MXFP4, requires sm_90+ for native speed — practical AWS = p5+).

## What we still don't know (the wiki's data gaps)

1. **Qwen2.5-Coder family BFCL scores** — model is absent from BFCL despite strong tool-calling claims in the tech report. Either evaluate it ourselves or wait for Qwen team submission.
2. **SWE-bench Verified scores for any of the 13 candidates** — none are published. Sub-30B dense models typically score <15% with stock harnesses.
3. **AWQ INT4 quantization impact on BFCL / SWE-bench / LCB** — no public AWQ-quantized eval runs in this domain. Aider Ollama Q8 numbers are the only quantized public references.
4. **DeepSeek-Coder-V2-Lite tool-calling parser** — `deepseek_v3` is V3-tuned; V2 behavior unverified.
5. **Granite 3.x BFCL** — IBM claims best-in-class but publishes no number for 3.1 / 3.3.
6. **Mistral-Small 3.2 BFCL** — qualitative "agentic" claim; no number.
7. **Hermes-3 BFCL** — community ~91% figure is well-formed-JSON rate, not BFCL accuracy.

## Related
- [[wiki/overview]]
- [[concepts/tool-selection]], [[concepts/code-generation]], [[concepts/benchmarks]]
- [[hardware/a10g-g5xlarge]], [[hardware/multi-gpu-options]]
- [[infrastructure/vllm]], [[infrastructure/nvidia-dynamo]], [[infrastructure/quantization]]

## Sources
- [[sources/aws-ec2-pricing-2026-05]]
- [[sources/nvidia-a10g-specs]]
- [[sources/nvidia-dynamo-readme]]
- [[sources/vllm-tool-calling-docs]]
- [[sources/vllm-quantization-docs]]
- [[sources/bfcl-leaderboard-2026-05]]
- [[sources/code-benchmarks-2026-05]]
- [[sources/qwen2.5-coder-tech-report]]
- [[sources/deepseek-coder-v2-paper]]
- [[sources/codestral-mnpl-license]]
- [[sources/llama-3.x-cards]]
- [[sources/mistral-small-3.x-cards]]
- [[sources/phi-3-and-phi-4-cards]]
- [[sources/granite-3.x-cards]]
- [[sources/hermes-3-and-4-cards]]
- [[sources/xlam-family-cards]]
- [[sources/functionary-readme]]
