---
tags: [overview]
last_updated: 2026-05-07
source_count: 0
---

# Overview

The research question: **which open-source LLMs are credible candidates for tool selection and complex code generation, when served via NVIDIA Dynamo + vLLM on AWS EC2 g5.xlarge (single NVIDIA A10G, 24 GB VRAM)?**

## Hard constraints

- **GPU**: single NVIDIA A10G, 24 GB GDDR6, sm_86 (Ampere) — no native FP8 tensor cores. See [[hardware/a10g-g5xlarge]].
- **Instance**: AWS EC2 g5.xlarge as the primary target. Multi-GPU options (g5.12xlarge, g5.48xlarge, p4d, p5) are catalogued in [[hardware/multi-gpu-options]] for models that don't fit.
- **Serving stack**: [[infrastructure/vllm]] under [[infrastructure/nvidia-dynamo]]. Models must be supported by vLLM and have a working tool-call parser.
- **Capability focus**: [[concepts/tool-selection]] and [[concepts/code-generation]]. Pure conversational quality is secondary.

## What "wins" looks like

A model wins if it:
1. **Fits** at a usable context length (≥ 8k effective KV cache) on a single A10G, ideally at AWQ INT4 or INT8.
2. **Scores well** on the relevant benchmarks: BFCL for tool calling, HumanEval/MBPP/LiveCodeBench/SWE-bench for code. See [[concepts/benchmarks]].
3. **Has a vLLM tool-call parser** (or works with the generic parser well enough). See [[infrastructure/vllm]].
4. **Costs reasonably**: g5.xlarge is ~$1.006/hr on-demand, ~$734/month 24×7 *(unverified — needs source)*. Spot can cut this 30–70%.

## Candidate landscape

Three buckets (full list in [[index]]):

- **Tool-calling specialists** — fine-tuned to be good at function calling: [[models/xlam-7b]], [[models/functionary-small]], [[models/hermes-3-llama-3.1-8b]].
- **Code specialists** — strong on code benchmarks; tool calling varies: [[models/qwen2.5-coder-7b]], [[models/qwen2.5-coder-14b]], [[models/qwen2.5-coder-32b]], [[models/codestral-22b]], [[models/deepseek-coder-v2-lite]].
- **Generalists with strong tool use** — solid all-around with first-class tool calling: [[models/llama-3.1-8b-instruct]], [[models/llama-3.3-70b-instruct]] (multi-GPU only), [[models/mistral-small-24b]], [[models/granite-3.1-8b]], [[models/phi-3-medium-14b]].

The artifact that answers the research question end-to-end is the master table at [[comparisons/tool-calling-models-on-a10g]].

## Open questions

- Which models have BFCL v3 (multi-turn) scores published, vs only v1/v2?
- Does Dynamo's disaggregated prefill/decode help on a single GPU at all, or is it strictly multi-GPU value?
- For 32B-class models, is AWQ INT4 on A10G actually competitive in latency vs an FP16 14B model on the same card?
- How much KV-cache overhead does Qwen2.5-Coder's 128k context window cost in practice — what's the largest "real" context we can afford on 24 GB?
- For repo-scale tasks (SWE-bench-style), do any sub-32B open models actually clear single-digit %? Where does the open/closed gap stand as of mid-2026?

## Sources
- (none ingested yet)

## TODO / verify
- AWS EC2 g5 pricing page
- Berkeley Function-Calling Leaderboard (BFCL) v3 snapshot
- vLLM tool-call parser docs (latest release)
- NVIDIA Dynamo architecture docs
- Hugging Face model cards for every candidate
