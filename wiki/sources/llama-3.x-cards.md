---
tags: [source, models, llama, meta]
source_path: raw/llama-3.1-8b-hf-card.md, raw/llama-3.3-70b-hf-card.md, raw/llama-3.1-community-license.md
source_url: https://huggingface.co/meta-llama/Meta-Llama-3.1-8B-Instruct, https://huggingface.co/meta-llama/Llama-3.3-70B-Instruct
ingested: 2026-05-07
last_updated: 2026-05-07
---

# Meta Llama 3.1 / 3.3 — model cards & licenses

## Key claims

### Llama 3.1 8B Instruct
- **HF model ID**: `meta-llama/Meta-Llama-3.1-8B-Instruct`
- **Params**: 8B dense
- **Context**: 128K
- **Released**: 2024-07-23 (knowledge cutoff Dec 2023)
- **License**: Llama 3.1 Community License — commercial use allowed with attribution. **700M MAU clause**: any product/service exceeding 700M MAU on the release date must request a separate license from Meta. "Built with Llama" attribution required for derivatives; derived models must include "Llama" in their name. Acceptable Use Policy applies.

#### Benchmarks (Meta self-report, eval_details)
- HumanEval: **72.6**
- MBPP++ base: 72.8
- BFCL v2: **76.1**
- MMLU 5-shot: 69.4
- MMLU-Pro CoT: 48.3
- MATH: 51.9
- GSM-8K: 84.5

#### vLLM serving
- `--tool-call-parser llama3_json --chat-template examples/tool_chat_template_llama3.1_json.jinja`
- ⚠ **Parallel tool calls not supported** for Llama 3.x (only Llama 4 adds them).
- Pre-quantized AWQ/GPTQ widely available (e.g. `hugging-quants/Meta-Llama-3.1-8B-Instruct-AWQ-INT4`).

### Llama 3.3 70B Instruct
- **HF model ID**: `meta-llama/Llama-3.3-70B-Instruct`
- **Params**: 70B dense
- **Context**: 128K
- **Released**: 2024-12-06
- **License**: Llama 3.3 Community License Agreement — same structure and 700M MAU clause as 3.1.

#### Benchmarks (Meta self-report)
- HumanEval: **88.4** (vs 80.5 for 3.1 70B)
- MBPP EvalPlus base: 87.6
- BFCL v2: **77.3** (overall_ast_summary / macro_avg / valid)
- MMLU CoT 0-shot: 86.0
- MMLU Pro 5-shot CoT: 68.9
- IFEval: 92.1
- MATH: 77.0
- GPQA Diamond: 50.5
- MGSM: 91.1

#### vLLM serving
- Same parser as 3.1: `--tool-call-parser llama3_json --chat-template examples/tool_chat_template_llama3.1_json.jinja`
- Common quants: `neuralmagic/Llama-3.3-70B-Instruct-FP8-dynamic`, `hugging-quants/Meta-Llama-3.3-70B-Instruct-AWQ-INT4`
- Multi-GPU only on AWS (won't fit single A10G in any quant with usable context).

### Successor status

- Meta has not released a Llama-4 model in the 8B-dense slot. **Llama-3.1-8B remains Meta's current 8B dense instruct.**
- Llama 4 Scout (17B active / 16-expert MoE, ~109B total) is the smallest Llama 4 — does not fit single A10G unquantized.
- **Llama-3.3-70B remains Meta's flagship dense 70B-class instruct.** Llama 4 Scout/Maverick are MoE multimodal, not direct dense replacements.

## Citation URLs
- [HF: Meta-Llama-3.1-8B-Instruct](https://huggingface.co/meta-llama/Meta-Llama-3.1-8B-Instruct)
- [HF: Llama-3.3-70B-Instruct](https://huggingface.co/meta-llama/Llama-3.3-70B-Instruct)
- [Llama 3.1 Community License](https://github.com/meta-llama/llama-models/blob/main/models/llama3_1/LICENSE)
- [Llama 3.3 Community License](https://github.com/meta-llama/llama-models/blob/main/models/llama3_3/LICENSE)
- [Meta-llama eval_details (Llama 3.1)](https://github.com/meta-llama/llama-models/blob/main/models/llama3_1/eval_details.md)
- [Llama 4 announcement](https://ai.meta.com/blog/llama-4-multimodal-intelligence/)

## Pages updated on ingest
- [[models/llama-3.1-8b-instruct]]
- [[models/llama-3.3-70b-instruct]]
- [[models/hermes-3-llama-3.1-8b]] (base model)
- [[wiki/comparisons/tool-calling-models-on-a10g]]
