# Index

Catalog of every page in the wiki. Update on every ingest. See [[AGENTS]] for conventions.

## Overview
- [[wiki/overview]] — research question, hard constraints, success criteria, recommended defaults

## Hardware
- [[wiki/hardware/aws-gpu-landscape]] — **master GPU/instance menu** (g4dn / g5 / g6 / g6e / p4 / p5 / p6) with pricing, sm levels, FP8/MXFP4 compatibility
- [[wiki/hardware/a10g-g5xlarge]] — A10G specs (70 TF FP16 dense, 600 GB/s, sm_86), g5.xlarge $1.006/hr, VRAM budget math
- [[wiki/hardware/g6e-l40s]] — L40S 48 GB single-GPU deep dive; g6e.xlarge $1.861/hr — cheapest 48 GB box on AWS
- [[wiki/hardware/multi-gpu-options]] — multi-GPU decision logic (TP/PP/EP), model-size → smallest single-node AWS box

## Infrastructure
- [[wiki/infrastructure/vllm]] — vLLM 0.20.x features for tool calling and code; full parser table
- [[wiki/infrastructure/nvidia-dynamo]] — what Dynamo adds (and why it's near-zero on a single GPU)
- [[wiki/infrastructure/quantization]] — AWQ / GPTQ / FP8 / INT8 / GGUF / MXFP4 on Ampere/Ada/Hopper/Blackwell

## Concepts
- [[wiki/concepts/tool-selection]] — function calling, parallel tools, BFCL sub-skills
- [[wiki/concepts/code-generation]] — single-function vs repo-scale; current open-source state
- [[wiki/concepts/benchmarks]] — BFCL, HumanEval+/MBPP+, LiveCodeBench, SWE-bench Verified, Aider, τ-bench

## Models — A10G class (≤ 24 GB single GPU)
- [[wiki/models/qwen2.5-coder-7b]] — Apache-2.0; HE 88.4 / LCB 18.2; sweet 7B code model — `hermes` parser flaky
- [[wiki/models/qwen2.5-coder-14b]] — Apache-2.0; HE 89.6 / LCB 23.4 — superseded by Qwen3-Coder-30B-A3B
- [[wiki/models/qwen2.5-coder-32b]] — Apache-2.0; HE 92.7 / LCB 31.4; tight A10G fit at AWQ INT4
- [[wiki/models/qwen3-coder-30b-a3b]] — **NEW** Apache-2.0 MoE 31B/3.3B; mainline `qwen3_coder` parser; SWE-V ~50.3
- [[wiki/models/qwen3-32b]] — **NEW** Apache-2.0 dense; hybrid reasoning; AWQ INT4 fits but tight ctx
- [[wiki/models/devstral-small]] — **NEW** Apache-2.0 24B; SWE-V 53.6 → 68.0; replaces Codestral-22B for commercial use
- [[wiki/models/llama-3.1-8b-instruct]] — Llama community license; HE 72.6 / **BFCL V2 76.1**; baseline generalist
- [[wiki/models/deepseek-coder-v2-lite]] — DeepSeek License (commercial OK); 16B MoE / 2.4B active; HE 81.1 / LCB 24.3
- [[wiki/models/mistral-small-24b]] — Apache-2.0; canonical: Mistral-Small-3.2-24B-Instruct-2506
- [[wiki/models/codestral-22b]] — **MNPL — non-commercial**; superseded by Devstral-Small for commercial use
- [[wiki/models/phi-3-medium-14b]] — MIT; HE 58.5; **no tool-call parser — disqualified**
- [[wiki/models/granite-3.1-8b]] — Apache-2.0; canonical: Granite-3.3-8B-Instruct, HE 89.73; first-class `granite` parser
- [[wiki/models/hermes-3-llama-3.1-8b]] — Llama community; canonical `hermes` parser target
- [[wiki/models/xlam-7b]] — **CC-BY-NC-4.0 — research only**; v2 8B is `Llama-xLAM-2-8b-fc-r`
- [[wiki/models/functionary-small]] — MIT; functionary-small-v3.2 BFCL 82.82; **no mainline vLLM parser** — needs MeetKai fork

## Models — Multi-GPU / frontier
- [[wiki/models/llama-3.3-70b-instruct]] — Llama community; HE 88.4 / BFCL V2 77.3; multi-GPU only
- [[wiki/models/glm-4.5-air]] — **NEW** MIT 106B/12B MoE; parent GLM-4.5 SWE-V 64.2 / TAU 70.1 (Air-specific unpublished); vLLM `glm45` parser
- [[wiki/models/llama-4-scout]] — **NEW** Llama 4 Community 109B/17B MoE; 10M ctx; multimodal
- [[wiki/models/qwen3-coder-480b]] — **NEW** Apache-2.0 480B/35B MoE; SWE-V 66.5; flagship Apache code
- [[wiki/models/deepseek-v3.1]] — **NEW** MIT 671B/37B MoE; SWE-V 66.0; native FP8
- [[wiki/models/kimi-k2]] — **NEW** mod-MIT 1T/32B MoE; K2.6 SWE-V **80.2** — open-source frontier
- [[wiki/models/gpt-oss-20b]] — **NEW** Apache-2.0 21B/3.6B MXFP4; SWE-V 60.7; needs sm_90+ for native MXFP4
- [[wiki/models/gpt-oss-120b]] — **NEW** Apache-2.0 117B/5.1B MXFP4; o3-mini class

## Comparisons
- [[wiki/comparisons/tool-calling-models-on-a10g]] — original master table: A10G-locked
- [[wiki/comparisons/models-by-budget]] — **NEW** budget-tiered: best model per $/hr bracket across the full AWS GPU spectrum

## Sources
- [[wiki/sources/aws-ec2-pricing-2026-05]] — AWS G5/P4/P5 on-demand pricing, us-east-1, May 2026
- [[wiki/sources/aws-extended-gpu-pricing-2026-05]] — **NEW** full G4dn/G5/G6/G6e/P4/P5/P6 pricing + sm levels
- [[wiki/sources/nvidia-a10g-specs]] — NVIDIA A10/A10G datasheet diff
- [[wiki/sources/nvidia-l4-l40s-specs]] — **NEW** Ada Lovelace L4 / L40S datasheets
- [[wiki/sources/nvidia-dynamo-readme]] — Dynamo features and explicit single-GPU disclaimer
- [[wiki/sources/vllm-tool-calling-docs]] — full vLLM tool-call parser table + per-model recommendations
- [[wiki/sources/vllm-quantization-docs]] — AWQ/GPTQ/Marlin support on Ampere; FP8 W8A8; flag defaults
- [[wiki/sources/bfcl-leaderboard-2026-05]] — BFCL V1/V2/V3/V4 methodology + per-model scores
- [[wiki/sources/code-benchmarks-2026-05]] — SWE-bench Verified, LiveCodeBench, Aider, EvalPlus
- [[wiki/sources/qwen2.5-coder-tech-report]] — arXiv 2409.12186 + HF cards for 7B/14B/32B
- [[wiki/sources/qwen3-coder-cards]] — **NEW** Qwen3-Coder 30B-A3B + 480B-A35B HF cards + Qwen blog + vLLM recipe
- [[wiki/sources/qwen3-dense-cards]] — **NEW** Qwen3-32B / 14B / 8B dense (hybrid reasoning)
- [[wiki/sources/deepseek-coder-v2-paper]] — arXiv 2406.11931 + DeepSeek-Coder-V2-Lite-Instruct card
- [[wiki/sources/deepseek-v3-r1-family]] — **NEW** DeepSeek V3.1 + R1-0528 cards + tech reports
- [[wiki/sources/codestral-mnpl-license]] — Codestral 22B HF card + MNPL-0.1 license
- [[wiki/sources/devstral-small-cards]] — **NEW** Devstral-Small-2507 + Devstral-2-123B cards
- [[wiki/sources/llama-3.x-cards]] — Llama 3.1 8B + Llama 3.3 70B HF cards + community license
- [[wiki/sources/llama-4-cards]] — **NEW** Llama 4 Scout + Maverick cards + Meta announcement
- [[wiki/sources/mistral-small-3.x-cards]] — Mistral Small 24B 2501/3.1/3.2 cards
- [[wiki/sources/phi-3-and-phi-4-cards]] — Phi-3-medium-128k + Phi-4 14B cards
- [[wiki/sources/granite-3.x-cards]] — Granite 3.0/3.1/3.3 cards + IBM announcement
- [[wiki/sources/hermes-3-and-4-cards]] — Hermes-3-Llama-3.1-8B + Hermes-4 70B/405B cards
- [[wiki/sources/xlam-family-cards]] — Salesforce xLAM v1 + v2 cards + APIGen-MT paper
- [[wiki/sources/functionary-readme]] — MeetKai Functionary GitHub README + small-v3.2 card
- [[wiki/sources/glm-4.5-cards]] — **NEW** GLM-4.5 / GLM-4.5-Air HF cards + ARC paper
- [[wiki/sources/kimi-k2-cards]] — **NEW** Kimi-K2 / K2-Thinking / K2.5 / K2.6 cards
- [[wiki/sources/gpt-oss-cards]] — **NEW** OpenAI gpt-oss-20b / gpt-oss-120b cards + MXFP4 hardware note
