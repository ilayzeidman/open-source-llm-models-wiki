---
tags: [models, generalist, tool-calling, ibm]
params: 8B
active_params: 8B
license: Apache-2.0
context: 128k
release_date: 2024-12-18 (3.1); 2025-04-16 (3.3)
last_updated: 2026-05-07
source_count: 3
---

# Granite-3.x-8B-Instruct (canonical: 3.3)

IBM's 8B generalist with first-class tool-calling support and a dedicated vLLM `granite` parser. Apache-2.0 + IBM IP indemnity. **Granite 3.3 (April 2025) supersedes 3.1** on every benchmark — the wiki canonical recommendation is now `granite-3.3-8b-instruct`.

- 3.3: `ibm-granite/granite-3.3-8b-instruct` ([model card](https://huggingface.co/ibm-granite/granite-3.3-8b-instruct))
- 3.1: `ibm-granite/granite-3.1-8b-instruct` ([model card](https://huggingface.co/ibm-granite/granite-3.1-8b-instruct))
- Architecture: 8.1B dense (40 layers, 4096 embed, 32 heads / 8 KV heads, MLP 12800, SwiGLU, RoPE)

## Fit on A10G (24 GB)
- FP16: ~16 GB → ✅ ~6 GB KV budget
- AWQ INT4: ~5 GB → ✅ ~17 GB KV budget — long context feasible
- **Verdict**: ✅ comfortable on g5.xlarge

29–30 community quant variants per version (llama.cpp / LM Studio / Jan / Ollama). NVIDIA NIM hosts the 3.3 model. [Source: [[sources/granite-3.x-cards]]]

## Strengths (IBM model cards)

| Benchmark | 3.1-8B | 3.3-8B |
|---|---|---|
| HumanEval | not on card | **89.73** |
| HumanEval+ | not on card | 86.09 |
| IFEval | 72.08 | 74.82 |
| MMLU | 65.34 | 65.54 |
| MATH-500 | — | 69.02 |
| GSM8K | 73.84 | 80.89 |
| Arena-Hard | — | 57.56 |
| AlpacaEval-2.0 | — | 62.68 |
| AIME24 | — | 8.12 |

3.3 also adds **`<think>...</think>` / `<response>...</response>` reasoning tags** toggled via `tokenizer.apply_chat_template(..., thinking=True)`.

[Source: [[sources/granite-3.x-cards]], [[sources/code-benchmarks-2026-05]]]

## Tool calling

- vLLM parser: `--tool-call-parser granite` for both 3.1 and 3.3 (3.0 needed an explicit chat-template arg; 3.1+ ship the right template in the tokenizer).
- IBM's 3.0 announcement claims best-in-weight-class **BFCL** scores, but **no specific BFCL number is published** in the announcement or the 3.1 / 3.3 model cards.
- **Granite 4.1-8B-Instruct (newer line)** does report BFCL v3 = **68.27** per IBM blog, plus HumanEval ~87.2 — uses `--tool-call-parser granite4`.

[Source: [[sources/vllm-tool-calling-docs]], [[sources/bfcl-leaderboard-2026-05]]]

## Weaknesses
- 3.1 model card lacks HumanEval/MBPP/BFCL — IBM's own benchmark coverage is patchy. 3.3 fixes this.
- Less community tooling/finetune ecosystem than Llama/Qwen.
- Code score (89.73 HE on 3.3) is strong but trails [[models/qwen2.5-coder-7b]] (88.4) only marginally — and 3.3 is not a code specialist.

## vLLM serving notes
```
vllm serve ibm-granite/granite-3.3-8b-instruct \
  --tool-call-parser granite \
  --enable-auto-tool-choice
```
AWQ/GPTQ checkpoints available from community. [Source: [[sources/vllm-tool-calling-docs]]]

## Cost (AWS, on-demand, us-east-1)
- Smallest box: **g5.xlarge** ($1.006/hr)
- Among the cheapest tool-calling-capable models to serve.
- [Source: [[sources/aws-ec2-pricing-2026-05]]]

## Recommendation

For an Apache-2.0 commercial-friendly tool-caller on a single A10G with strong code performance, **Granite 3.3 8B is the leading IBM choice** — and arguably the leading enterprise-license choice overall. Watch Granite 4.x for newer-line replacements.

## Related
- [[models/llama-3.1-8b-instruct]] — comparable size, different family/license
- [[models/hermes-3-llama-3.1-8b]] — alt 8B with strong tool calling
- [[concepts/tool-selection]]
- [[wiki/comparisons/tool-calling-models-on-a10g]]

## Sources
- [[sources/granite-3.x-cards]]
- [[sources/vllm-tool-calling-docs]]
- [[sources/code-benchmarks-2026-05]]
