---
tags: [models, tool-calling, specialist, nous-research]
params: 8B
active_params: 8B
license: Llama 3.1 Community License
context: 128k
release_date: 2024-08-15
last_updated: 2026-05-07
source_count: 3
---

# Hermes-3-Llama-3.1-8B

NousResearch's tool-calling-focused fine-tune of [[models/llama-3.1-8b-instruct]]. **The canonical target for vLLM's `hermes` parser** — also reused by Qwen2.5/Qwen3 deployments.

- HF: `NousResearch/Hermes-3-Llama-3.1-8B` ([model card](https://huggingface.co/NousResearch/Hermes-3-Llama-3.1-8B))
- Tech report: [arXiv 2408.11857](https://arxiv.org/abs/2408.11857)
- Base: `meta-llama/Llama-3.1-8B`

## Fit on A10G (24 GB)
- FP16: ~16 GB → ✅ ~6 GB KV budget
- AWQ INT4: ~5 GB → ✅ huge KV budget
- **Verdict**: ✅ excellent A10G fit

GGUF mirror: `Hermes-3-Llama-3.1-8B-GGUF`. Larger Hermes-3 variants (70B, 405B) exist but don't fit single A10G.

## Strengths

- **Canonical Hermes tool-call format**: `<tool_call>{"name": ..., "arguments": ...}</tool_call>` JSON-in-XML inside ChatML system prompts. The `hermes` parser in vLLM is named after this model. [Source: [[sources/hermes-3-and-4-cards]], [[sources/vllm-tool-calling-docs]]]
- Strong system-prompt steerability (a Hermes signature).
- 128k context (inherited from Llama 3.1).
- vLLM mainline first-class support — no plugin or fork required.

## Weaknesses
- ⚠ **No official BFCL number on the card.** Community blogs report ~91% but that figure is the well-formed-JSON rate, **not BFCL overall accuracy** — don't quote as comparable. [Source: [[sources/bfcl-leaderboard-2026-05]]]
- Open LLM Leaderboard avg 23.49 (IFEval 61.70, BBH 30.72, MMLU-PRO 23.77) — modest general-knowledge floor.
- Code: not a code specialist; prefer [[models/qwen2.5-coder-7b]] for code-heavy work.
- Inherits Llama 3.1 Community License (commercial-OK with 700M MAU + AUP + "Built with Llama" attribution).

## vLLM serving notes
- `--tool-call-parser hermes`. No chat-template flag needed (uses tokenizer's built-in).
- Docs explicitly say "all Nous Hermes models newer than Hermes 2 Pro are supported." Hermes 2 Theta is flagged as degraded — avoid.
- AWQ/GPTQ widely available in community.

## Cost (AWS, on-demand, us-east-1)
- Smallest box: **g5.xlarge** ($1.006/hr). [Source: [[sources/aws-ec2-pricing-2026-05]]]

## Successor / current state

**Hermes-4** (Aug 2025) was released in **70B and 405B only — no 8B variant**. The 8B gap remains for A10G-class deployment. For the foreseeable future, Hermes-3-Llama-3.1-8B is **the only Nous tool-calling specialist that fits a single A10G**.

Hermes-4 adds hybrid reasoning mode (`<think>...</think>` blocks) — same tool-call format. If you have multi-GPU (g5.12xlarge / p4d), Hermes-4-70B is the upgrade path with `--tool-call-parser hermes`.

## Related
- [[models/llama-3.1-8b-instruct]] — base model
- [[models/xlam-7b]], [[models/functionary-small]] — other tool-calling specialists
- [[concepts/tool-selection]], [[infrastructure/vllm]]
- [[wiki/comparisons/tool-calling-models-on-a10g]]

## Sources
- [[sources/hermes-3-and-4-cards]]
- [[sources/vllm-tool-calling-docs]]
- [[sources/bfcl-leaderboard-2026-05]]
