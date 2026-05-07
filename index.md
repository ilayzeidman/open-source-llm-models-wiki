# Index

Catalog of every page in the wiki. Update on every ingest. See [[AGENTS]] for conventions.

## Overview
- [[wiki/overview]] — research question, hard constraints, success criteria, recommended defaults

## Hardware
- [[wiki/hardware/a10g-g5xlarge]] — A10G specs (70 TF FP16 dense, 600 GB/s, sm_86), g5.xlarge $1.006/hr, VRAM budget math
- [[wiki/hardware/multi-gpu-options]] — when to step up; full G5/P4d/P5 menu with verified pricing

## Infrastructure
- [[wiki/infrastructure/vllm]] — vLLM 0.20.1 features for tool calling and code; full parser table
- [[wiki/infrastructure/nvidia-dynamo]] — what Dynamo adds (and why it's near-zero on a single GPU)
- [[wiki/infrastructure/quantization]] — AWQ / GPTQ / FP8 / INT8 / GGUF on Ampere; Marlin yes, FP8 W8A8 no

## Concepts
- [[wiki/concepts/tool-selection]] — function calling, parallel tools, BFCL sub-skills
- [[wiki/concepts/code-generation]] — single-function vs repo-scale; current open-source state
- [[wiki/concepts/benchmarks]] — BFCL, HumanEval+/MBPP+, LiveCodeBench, SWE-bench Verified, Aider, τ-bench

## Models
- [[wiki/models/qwen2.5-coder-7b]] — Apache-2.0; HE 88.4 / LCB 18.2; sweet 7B code model — `hermes` parser flaky
- [[wiki/models/qwen2.5-coder-14b]] — Apache-2.0; HE 89.6 / LCB 23.4; **A10G code+tools sweet spot** — `hermes` parser flaky
- [[wiki/models/qwen2.5-coder-32b]] — Apache-2.0; HE 92.7 / LCB 31.4; tight A10G fit at AWQ INT4
- [[wiki/models/llama-3.1-8b-instruct]] — Llama community license; HE 72.6 / **BFCL V2 76.1**; baseline generalist
- [[wiki/models/llama-3.3-70b-instruct]] — Llama community license; HE 88.4 / BFCL V2 77.3; multi-GPU only
- [[wiki/models/deepseek-coder-v2-lite]] — DeepSeek License (commercial OK); 16B MoE / 2.4B active; HE 81.1 / LCB 24.3
- [[wiki/models/mistral-small-24b]] — Apache-2.0; **canonical: Mistral-Small-3.2-24B-Instruct-2506** with 128K and improved tool calling
- [[wiki/models/codestral-22b]] — **MNPL — non-commercial**; HE 86.6 / MBPP 91.2; not trained on tool-call format
- [[wiki/models/phi-3-medium-14b]] — MIT; HE 58.5; **no tool-call parser — agentic disqualified** (Phi-4 14B same issue)
- [[wiki/models/granite-3.1-8b]] — Apache-2.0; **canonical: Granite-3.3-8B-Instruct** with HE 89.73; first-class `granite` parser
- [[wiki/models/hermes-3-llama-3.1-8b]] — Llama community; canonical target for vLLM `hermes` parser; only Nous specialist that fits A10G
- [[wiki/models/xlam-7b]] — **CC-BY-NC-4.0 — research only**; v1 fc-r BFCL V1 88.24; v2 8B is `Llama-xLAM-2-8b-fc-r`
- [[wiki/models/functionary-small]] — MIT; functionary-small-v3.2 BFCL 82.82; **no mainline vLLM parser** — needs MeetKai fork

## Comparisons
- [[wiki/comparisons/tool-calling-models-on-a10g]] — master table: model × VRAM × benchmarks × license × parser × cost × verdict

## Sources
- [[wiki/sources/aws-ec2-pricing-2026-05]] — AWS G5/P4/P5 on-demand pricing, us-east-1, May 2026
- [[wiki/sources/nvidia-a10g-specs]] — NVIDIA A10/A10G datasheet diff (A10G = ~56% of A10's tensor TFLOPS)
- [[wiki/sources/nvidia-dynamo-readme]] — Dynamo features and explicit single-GPU disclaimer
- [[wiki/sources/vllm-tool-calling-docs]] — full vLLM 0.20.x tool-call parser table + per-model recommendations
- [[wiki/sources/vllm-quantization-docs]] — AWQ/GPTQ/Marlin support on Ampere; FP8 W8A8 unsupported; flag defaults
- [[wiki/sources/bfcl-leaderboard-2026-05]] — BFCL V1/V2/V3/V4 methodology + per-model scores for the 13 candidates
- [[wiki/sources/code-benchmarks-2026-05]] — SWE-bench Verified, LiveCodeBench, Aider, EvalPlus snapshot (May 2026)
- [[wiki/sources/qwen2.5-coder-tech-report]] — arXiv 2409.12186 + HF cards for 7B/14B/32B
- [[wiki/sources/deepseek-coder-v2-paper]] — arXiv 2406.11931 + DeepSeek-Coder-V2-Lite-Instruct card; license is commercial-OK
- [[wiki/sources/codestral-mnpl-license]] — Codestral 22B HF card + announcement + MNPL-0.1 license text
- [[wiki/sources/llama-3.x-cards]] — Llama 3.1 8B + Llama 3.3 70B HF cards + community license
- [[wiki/sources/mistral-small-3.x-cards]] — Mistral Small 24B 2501/3.1/3.2 HF cards + announcements
- [[wiki/sources/phi-3-and-phi-4-cards]] — Phi-3-medium-128k + Phi-4 14B HF cards
- [[wiki/sources/granite-3.x-cards]] — Granite 3.0/3.1/3.3 cards + IBM announcement
- [[wiki/sources/hermes-3-and-4-cards]] — Hermes-3-Llama-3.1-8B + Hermes-4 70B/405B cards
- [[wiki/sources/xlam-family-cards]] — Salesforce xLAM v1 + v2 cards + APIGen-MT paper + CC-BY-NC-4.0 terms
- [[wiki/sources/functionary-readme]] — MeetKai Functionary GitHub README + small-v3.2 card
