# Index

Catalog of every page in the wiki. Update on every ingest. See [[AGENTS]] for conventions.

## Overview
- [[wiki/overview]] — research question, hard constraints, success criteria

## Hardware
- [[wiki/hardware/a10g-g5xlarge]] — A10G specs, g5.xlarge pricing, VRAM budget math
- [[wiki/hardware/multi-gpu-options]] — when to step up from g5.xlarge; g5/p4d/p5 menu

## Infrastructure
- [[wiki/infrastructure/vllm]] — vLLM features relevant to tool calling and code
- [[wiki/infrastructure/nvidia-dynamo]] — what Dynamo adds on top of vLLM
- [[wiki/infrastructure/quantization]] — AWQ / GPTQ / FP8 / INT8 / GGUF tradeoffs on A10G

## Concepts
- [[wiki/concepts/tool-selection]] — function calling, parallel tools, irrelevance detection
- [[wiki/concepts/code-generation]] — single-function vs repo-scale code tasks
- [[wiki/concepts/benchmarks]] — BFCL, HumanEval, MBPP, SWE-bench, LiveCodeBench

## Models
- [[wiki/models/qwen2.5-coder-7b]] — Qwen team's 7B code model, strong tool calling
- [[wiki/models/qwen2.5-coder-14b]] — 14B code model, sweet spot for A10G with AWQ
- [[wiki/models/qwen2.5-coder-32b]] — 32B code model, tight fit at AWQ on A10G
- [[wiki/models/llama-3.1-8b-instruct]] — Meta's 8B generalist with native tool calling
- [[wiki/models/llama-3.3-70b-instruct]] — 70B generalist; multi-GPU only
- [[wiki/models/deepseek-coder-v2-lite]] — 16B MoE with 2.4B active, code specialist
- [[wiki/models/mistral-small-24b]] — Mistral Small 3 24B, generalist with tool calling
- [[wiki/models/codestral-22b]] — Mistral's code specialist
- [[wiki/models/phi-3-medium-14b]] — Microsoft's 14B; small footprint
- [[wiki/models/granite-3.1-8b]] — IBM's 8B with first-class tool calling
- [[wiki/models/hermes-3-llama-3.1-8b]] — NousResearch's tool-calling tune of Llama 3.1
- [[wiki/models/xlam-7b]] — Salesforce tool-calling specialist
- [[wiki/models/functionary-small]] — MeetKai's tool-calling specialist

## Comparisons
- [[wiki/comparisons/tool-calling-models-on-a10g]] — master table: model × VRAM × benchmarks × cost × verdict

## Sources
- (none ingested yet)
