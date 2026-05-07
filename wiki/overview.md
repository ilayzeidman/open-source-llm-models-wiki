---
tags: [overview]
last_updated: 2026-05-07
source_count: 4
---

# Overview

The research question: **which open-source LLMs are credible candidates for tool selection and complex code generation, when served via NVIDIA Dynamo + vLLM on AWS EC2 g5.xlarge (single NVIDIA A10G, 24 GB VRAM)?**

## Hard constraints

- **GPU**: single NVIDIA A10G, 24 GB GDDR6, sm_86 (Ampere) — no native FP8 tensor cores; ~70 TF FP16 dense (NOT 125 like the data-center A10). See [[hardware/a10g-g5xlarge]].
- **Instance**: AWS EC2 g5.xlarge as the primary target ($1.006/hr us-east-1 on-demand). Multi-GPU options catalogued in [[hardware/multi-gpu-options]].
- **Serving stack**: [[infrastructure/vllm]] (latest stable: 0.20.1) under [[infrastructure/nvidia-dynamo]]. Note: Dynamo's own README says use vLLM directly on a single GPU — Dynamo's value is multi-GPU/multi-node.
- **Capability focus**: [[concepts/tool-selection]] and [[concepts/code-generation]]. Pure conversational quality is secondary.

## What "wins" looks like

A model wins if it:
1. **Fits** at a usable context length (≥ 8k effective KV cache) on a single A10G, ideally at AWQ INT4 (Marlin kernels are excellent on Ampere; FP8 is not — see [[infrastructure/quantization]]).
2. **Scores well** on the relevant benchmarks: BFCL for tool calling, HumanEval/MBPP/LiveCodeBench/SWE-bench/Aider for code. See [[concepts/benchmarks]].
3. **Has a working vLLM tool-call parser** (mainline, not a fork). Models without one (Phi-3 / Phi-4, Functionary in mainline) are agentic-disqualified. See [[infrastructure/vllm]].
4. **Has a commercial-friendly license**. CC-BY-NC-4.0 (xLAM) and MNPL (Codestral 22B v0.1) are non-commercial blockers.
5. **Costs reasonably**: g5.xlarge is **$1.006/hr** on-demand, ~$734/month 24×7. Spot can cut this 30–70%. [Source: [[sources/aws-ec2-pricing-2026-05]]]

## Recommended defaults (verified May 2026)

For commercial deployment on g5.xlarge:

| Use case | Recommendation | Why |
|---|---|---|
| **Tool calling + reasoning** | [[models/granite-3.1-8b\|Granite-3.3-8B-Instruct]] | Apache-2.0, `granite` parser, HE 89.73 |
| **Code + tool calling** | [[models/qwen2.5-coder-14b\|Qwen2.5-Coder-14B-Instruct]] (AWQ) | Apache-2.0, sweet-spot fit, HE 89.6 / LCB 23.4 — note known parser bug |
| **Tool-calling specialist** | [[models/hermes-3-llama-3.1-8b\|Hermes-3-Llama-3.1-8B]] | Canonical `hermes` parser target |
| **Best generalist 24B** | [[models/mistral-small-24b\|Mistral-Small-3.2-24B-Instruct-2506]] | Apache-2.0, 128K, "agentic" tuning |

See [[wiki/comparisons/tool-calling-models-on-a10g]] for the master table.

## Candidate landscape (verified)

[Source: [[sources/bfcl-leaderboard-2026-05]], [[sources/code-benchmarks-2026-05]]]

- **Tool-calling specialists** — fine-tuned for function calling: [[models/hermes-3-llama-3.1-8b]] (Llama community, mainline parser), [[models/xlam-7b]] (CC-BY-NC, research only), [[models/functionary-small]] (MIT, no mainline parser).
- **Code specialists** — strong on code benchmarks: [[models/qwen2.5-coder-7b]], [[models/qwen2.5-coder-14b]], [[models/qwen2.5-coder-32b]] (Apache-2.0), [[models/codestral-22b]] (MNPL — non-commercial), [[models/deepseek-coder-v2-lite]] (DeepSeek License — commercial OK, MoE 16B/2.4B active).
- **Generalists with strong tool use**: [[models/llama-3.1-8b-instruct]] (BFCL v2 76.1), [[models/llama-3.3-70b-instruct]] (BFCL v2 77.3 — multi-GPU only), [[models/mistral-small-24b]] (Mistral Small 3.2), [[models/granite-3.1-8b]] (now Granite 3.3 with HE 89.73), [[models/phi-3-medium-14b]] (no tool calling — disqualified).

## Pitfalls and caveats discovered during ingest

1. **Qwen2.5-Coder is absent from the BFCL leaderboard** despite tool-calling claims in the tech report. The vLLM `hermes` parser also mis-extracts its output (model emits ```json``` fences instead of `<tool_call>` tags). Use the community parser plugin for production.
2. **All public BFCL/SWE-bench/LCB scores are FP16/BF16** — no public AWQ INT4 evaluations. Aider Ollama Q8 entries are the only quantized public references.
3. **SWE-bench Verified is not published** for any of the 13 wiki candidate models. Sub-30B dense models typically score <15% with stock harnesses; this benchmark is not currently a discriminator below ~30B.
4. **Mistral-Small versioning is messy** — three 24B releases in 13 months (2501 → 3.1 → 3.2). Pin specific dated revisions in production.
5. **Granite 3.3 supersedes 3.1** — HumanEval jumped from "not on card" to 89.73. Wiki canonical is now 3.3.
6. **Phi-3 and Phi-4 14B both lack vLLM tool parsers** — Phi-4-mini (3.8B) is the only Phi family member with native tool calling.
7. **Functionary needs MeetKai's `server_vllm.py` fork** — no mainline parser despite an active 8B variant.
8. **NVIDIA Dynamo offers essentially nothing for a single A10G** — its own README says use vLLM directly. The features (disaggregated prefill/decode, smart router, NIXL) are multi-GPU/multi-node by design.
9. **No NVLink on g5** — multi-GPU on g5.12xlarge / g5.48xlarge is over PCIe Gen4. For NVLink + FP8, step to p4d / p5.

## Open questions still worth investigating

- **Qwen3-Coder family** (Feb 2026) — does it actually deliver >70% SWE-bench Verified? When does an A10G-fitting variant land?
- **Real measured AWQ INT4 throughput on A10G** for each candidate — no public per-model numbers.
- **τ-bench** scores for Hermes-3-8B / Granite-3.3 / Mistral-Small-3.2 — closer to real agentic performance than BFCL.
- **Hermes-4-8B**: is one coming? The 8B gap is the only friction in the Nous lineup for A10G deployment.

## Sources
- [[sources/aws-ec2-pricing-2026-05]]
- [[sources/nvidia-a10g-specs]]
- [[sources/nvidia-dynamo-readme]]
- [[sources/vllm-tool-calling-docs]]
- [[sources/vllm-quantization-docs]]
- [[sources/bfcl-leaderboard-2026-05]]
- [[sources/code-benchmarks-2026-05]]
