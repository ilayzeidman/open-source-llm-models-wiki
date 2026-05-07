---
tags: [models, generalist, tool-calling, meta]
params: 8B
active_params: 8B
license: Llama 3.1 Community License
context: 128k
release_date: 2024-07-23
last_updated: 2026-05-07
source_count: 3
---

# Llama-3.1-8B-Instruct

Meta's 8B generalist with first-class tool-calling support. Widely deployed baseline; strong vLLM integration. Not a code specialist but solid all-around. Knowledge cutoff Dec 2023.

- HF: `meta-llama/Meta-Llama-3.1-8B-Instruct` ([model card](https://huggingface.co/meta-llama/Meta-Llama-3.1-8B-Instruct))

## Fit on A10G (24 GB)
- FP16/BF16 weights: ~16 GB → ✅ fits with ~6 GB KV-cache budget; OK for short context
- AWQ INT4: ~5 GB → ✅ huge KV budget; 100k+ ctx feasible at batch=1
- INT8: ~8 GB → ✅ comfortable
- **Verdict**: ✅ ideal A10G citizen

Common community quants: `hugging-quants/Meta-Llama-3.1-8B-Instruct-AWQ-INT4`, FP8 variants. [Source: [[sources/llama-3.x-cards]]]

## Strengths (Meta self-report, eval_details)

- Tool calling: **BFCL v2 76.1** [Source: [[sources/llama-3.x-cards]], [[sources/bfcl-leaderboard-2026-05]]]
- HumanEval: 72.6
- MBPP++ base: 72.8
- MMLU 5-shot: 69.4
- MMLU-Pro CoT: 48.3
- MATH: 51.9
- GSM-8K: 84.5
- Native 128k context, native tool support, well-supported parser in vLLM.

## Weaknesses
- Code: HumanEval 72.6 — well behind code specialists like [[models/qwen2.5-coder-7b]] (88.4) at the same parameter count.
- **No published SWE-bench Verified** or **LiveCodeBench** number.
- Llama 3.1 Community License (not Apache) — has commercial restrictions: 700M MAU clause, "Built with Llama" attribution required, derived models must include "Llama" in their name. [Source: [[sources/llama-3.x-cards]]]
- ⚠ **Parallel tool calls not supported** for Llama 3.x in vLLM (Llama 4 added them).

## vLLM serving notes
- Tool-call parser: `--tool-call-parser llama3_json --chat-template examples/tool_chat_template_llama3.1_json.jinja`
- Chat template ships with the model.
- Pre-quantized AWQ/GPTQ widely available.
- [Source: [[sources/vllm-tool-calling-docs]]]

## Cost (AWS, on-demand, us-east-1)
- Smallest box: **g5.xlarge** ($1.006/hr, ~$734/mo)
- High throughput at INT4 — among the cheapest models to serve.
- [Source: [[sources/aws-ec2-pricing-2026-05]]]

## Successor / current state

Meta has not released a Llama-4 model in the 8B-dense slot. Llama 4 Scout (17B active / 16-expert MoE, ~109B total) is the smallest Llama 4 — does not fit single A10G unquantized. **Llama-3.1-8B remains Meta's current 8B dense instruct.** For tool-calling specialization, see [[models/hermes-3-llama-3.1-8b]] (built on this base).

## Related
- [[models/llama-3.3-70b-instruct]] — bigger sibling, multi-GPU only
- [[models/hermes-3-llama-3.1-8b]] — tool-calling fine-tune of this base
- [[concepts/tool-selection]], [[infrastructure/vllm]]
- [[wiki/comparisons/tool-calling-models-on-a10g]]

## Sources
- [[sources/llama-3.x-cards]]
- [[sources/vllm-tool-calling-docs]]
- [[sources/bfcl-leaderboard-2026-05]]
