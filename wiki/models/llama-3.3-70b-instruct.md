---
tags: [models, generalist, tool-calling, meta, multi-gpu]
params: 70B
active_params: 70B
license: Llama 3.3 Community License
context: 128k
release_date: 2024-12-06
last_updated: 2026-05-07
source_count: 3
---

# Llama-3.3-70B-Instruct

Meta's 70B model, released Dec 2024. Closes most of the gap to Llama 3.1 405B at much lower cost. Strong tool calling and reasoning. **Does not fit on a single A10G** in any quantization with usable context.

- HF: `meta-llama/Llama-3.3-70B-Instruct` ([model card](https://huggingface.co/meta-llama/Llama-3.3-70B-Instruct))

## Fit on A10G (24 GB)
- FP16: ~140 GB → ❌
- INT8: ~70 GB → ❌
- AWQ INT4: ~40 GB → ❌
- **Verdict**: ❌ multi-GPU only. See [[hardware/multi-gpu-options]].

## Multi-GPU fit
- **g5.48xlarge** (8× A10G = 192 GB): ✅ AWQ INT4 with TP=8; ✅ INT8 with TP=8; FP16 tight
- **p4d.24xlarge** (8× A100 40 GB = 320 GB): ✅ FP16 with TP=8 (NVLink — best latency)
- **p5.48xlarge** (8× H100 80 GB = 640 GB): ✅ FP16 with TP=8; ✅ FP8 native

## Strengths (Meta self-report)

- Tool calling: **BFCL v2 77.3** (overall_ast_summary / macro_avg / valid)
- HumanEval: **88.4** (vs 80.5 for Llama 3.1 70B)
- MBPP EvalPlus base: 87.6
- MMLU CoT 0-shot: 86.0
- MMLU Pro 5-shot CoT: 68.9
- IFEval: 92.1
- MATH: 77.0
- GPQA Diamond: 50.5
- MGSM: 91.1

[Source: [[sources/llama-3.x-cards]], [[sources/bfcl-leaderboard-2026-05]], [[sources/code-benchmarks-2026-05]]]

## Weaknesses
- Cost: g5.48xlarge ~$16.288/hr ~$11,900/mo on-demand. [Source: [[sources/aws-ec2-pricing-2026-05]]]
- Latency on g5 (PCIe TP=8) is meaningfully worse than p4d (NVLink) — see [[hardware/multi-gpu-options]].
- Llama 3.3 Community License — same 700M MAU clause and attribution requirements as Llama 3.1.
- ⚠ **Parallel tool calls not supported** (Llama 3.x limitation).

## vLLM serving notes
- `--tool-call-parser llama3_json --chat-template examples/tool_chat_template_llama3.1_json.jinja --tensor-parallel-size 8`
- Use AWQ INT4 to fit on g5.48xlarge with comfortable KV budget.
- Common quants: `neuralmagic/Llama-3.3-70B-Instruct-FP8-dynamic` (Hopper-only), `hugging-quants/Meta-Llama-3.3-70B-Instruct-AWQ-INT4` (works on Ampere).

## Cost (AWS, on-demand, us-east-1)
- **g5.48xlarge**: $16.288/hr, ~$11,900/mo (cheapest fit on Ampere)
- **p4d.24xlarge**: $32.7726/hr — better latency, NVLink
- **p5.48xlarge**: $98.32/hr — best latency, FP8 native

## Successor / current state

Llama 4 Scout / Maverick (April 2025) are MoE multimodal models — not direct dense replacements. **Llama-3.3-70B remains Meta's flagship dense 70B-class instruct model** as of mid-2026. For an A10G-targeted wiki this is a multi-GPU model regardless.

## Related
- [[models/llama-3.1-8b-instruct]] — small sibling for single-A10G
- [[hardware/multi-gpu-options]]
- [[infrastructure/quantization]]
- [[wiki/comparisons/tool-calling-models-on-a10g]]

## Sources
- [[sources/llama-3.x-cards]]
- [[sources/bfcl-leaderboard-2026-05]]
- [[sources/aws-ec2-pricing-2026-05]]
