---
tags: [models, generalist, microsoft]
params: 14B
active_params: 14B
license: MIT
context: 128k
release_date: 2024-05-21
last_updated: 2026-05-07
source_count: 3
---

# Phi-3-Medium-128k-Instruct (and Phi-4 14B for context)

Microsoft's 14B Phi-3 generalist trained on heavily curated synthetic + filtered web data. **MIT license** is unusually permissive. Strong on reasoning per parameter; **tool calling is NOT supported** in vLLM mainline — major disqualifier for an agentic wiki.

- HF: `microsoft/Phi-3-medium-128k-instruct` ([model card](https://huggingface.co/microsoft/Phi-3-medium-128k-instruct))

## Fit on A10G (24 GB)
- FP16: ~28 GB → ❌
- INT8: ~14 GB → ✅ ~8 GB KV budget
- AWQ INT4: ~8–9 GB → ✅ comfortable, ~14 GB KV budget
- **Verdict**: ✅ at AWQ INT4 — but see tool-calling disqualifier below

Microsoft publishes ONNX variants (int4 DML, int4 CUDA, int4 CPU/Mobile, fp16 CUDA); many community AWQ/GGUF.

## Strengths (Microsoft model card)
- HumanEval 0-shot: 58.5
- MBPP 3-shot: 73.8
- MMLU 5-shot: 76.6
- GSM8K: 87.5
- BBH: 77.9
- MIT license, 128k context

[Source: [[sources/phi-3-and-phi-4-cards]]]

## Weaknesses (the agentic disqualifier)
- ⚠ **No native tool-call parser support in vLLM.** Model card does not document a tool template; vLLM has no `phi3_*` parser. Tool use requires prompt-based hacks or fine-tunes — **not first-class**. [Source: [[sources/vllm-tool-calling-docs]]]
- **No published BFCL, SWE-bench, LiveCodeBench, or Aider** scores.
- Code: HumanEval 58.5 — below all other 14B+ candidates in this wiki.
- Quality occasionally fragile to prompt format (Phi quirk).
- Successor models (Phi-4) supersede this on reasoning but **also lack tool calling** (see below).

## Phi-4 14B for context (also lacks tool calling)

`microsoft/phi-4` (Dec 2024) is the current Microsoft 14B-class flagship:

- HumanEval 82.6, MMLU 84.8, MATH 80.4, GPQA 56.1
- **BFCL v3 40.8** (per pricepertoken mirror, 2026-05-07)
- **Context dropped to 16K** from Phi-3-medium's 128K — bad for agentic
- ⚠ **No native tool calling.** vLLM has no `phi4_*` parser. Phi-4-mini (3.8B) added native function calling but it's a separate, smaller model.

[Source: [[sources/phi-3-and-phi-4-cards]]]

For agentic / tool-calling research on a single A10G, **neither Phi-3-medium nor Phi-4 14B is competitive with Llama-3.1-8B / Granite-3.3-8B / Mistral-Small-3.2** despite their strong reasoning numbers.

## vLLM serving notes
- Phi-3 supported in vLLM; chat template must match the model's expected format.
- No dedicated tool-call parser — rely on constrained decoding for any JSON tool calls (and expect brittleness).

## Cost (AWS, on-demand, us-east-1)
- Smallest box: **g5.xlarge** ($1.006/hr). [Source: [[sources/aws-ec2-pricing-2026-05]]]

## Recommendation for this wiki

**Skip Phi-3-medium and Phi-4 14B for tool-calling research.** They're great reasoning models with permissive licenses, but the missing tool-call parser is a deal-breaker for the wiki's primary research question. Use [[models/granite-3.1-8b]] (and Granite 3.3 specifically) or [[models/llama-3.1-8b-instruct]] for an Apache/MIT-friendly tool-calling default.

## Related
- [[concepts/tool-selection]] — note the parser-gap caveat
- [[infrastructure/vllm]]
- [[wiki/comparisons/tool-calling-models-on-a10g]]

## Sources
- [[sources/phi-3-and-phi-4-cards]]
- [[sources/vllm-tool-calling-docs]]
- [[sources/aws-ec2-pricing-2026-05]]
