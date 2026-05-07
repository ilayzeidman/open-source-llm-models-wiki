# DeepSeek-Coder-V2-Lite-Instruct — Hugging Face model card

source_url: https://huggingface.co/deepseek-ai/DeepSeek-Coder-V2-Lite-Instruct
fetched: 2026-05-07

## Identity
- HF model ID: `deepseek-ai/DeepSeek-Coder-V2-Lite-Instruct`
- Companion repo: https://github.com/deepseek-ai/DeepSeek-Coder-V2
- Paper: arXiv 2406.11931 (submitted 2024-06-17)

## Architecture / parameters
- Type: MoE (Mixture-of-Experts), DeepSeekMoE framework (arXiv 2401.06066)
- Total parameters: 16B
- Active parameters per token: 2.4B
- Context length: 128K tokens
- Larger sibling: DeepSeek-Coder-V2 (236B total, 21B active)

## Training
- Continued pretrain over DeepSeek-V2: 6T additional tokens (Lite total 10.2T = 4.2T base + 6T new)
- 338 programming languages (up from 86)

## License
- License name: "deepseek-license" (DeepSeek Model License — modified OpenRAIL-style, custom)
- Code: MIT
- Commercial use: explicitly permitted ("DeepSeek-Coder-V2 series (including Base and Instruct) supports commercial use.")
- License URL: https://github.com/deepseek-ai/DeepSeek-V2/blob/main/LICENSE-MODEL (V2 family text; identical/very similar agreement applies to V2 Coder)
- Restrictions: standard responsible-AI-style use restrictions (no illegal/harmful applications listed in license attachment); no field-of-use carve-outs blocking general commercial use.

## Benchmark scores (per DeepSeek-Coder-V2 paper / official repo, Lite-Instruct row)
- HumanEval (Python): 81.1
- MBPP+: 68.8
- LiveCodeBench (overall): 24.3
- USACO: 6.5
- FIM (Mean): 86.4
- Defects4J: 9.2
- SWE-bench: 0.0 *(reported as 0 — Lite is too small for the SWE-bench harness; full V2 scores meaningfully)*
- Aider: 44.4
- BBH (3-shot): 61.2
- MMLU (5-shot): 60.1
- ARC-Easy: 88.9 / ARC-Challenge: 77.4
- GSM8K: 86.4
- MATH: 61.8
- Arena-Hard: 38.1
- BFCL: not officially reported in paper

## Comparison context
- Lite-Instruct surpasses CodeLlama-33B / DeepSeek-Coder-33B on HumanEval despite 2.4B active params.
- Full DS-Coder-V2 (236B/21B): HE 90.2, MBPP+ 76.2, LCB 43.4 — much higher.

## Chat template
```
<｜begin▁of▁sentence｜>{system}

User: {user}

A: {assistant}<｜end▁of▁sentence｜>
```
DeepSeek's standard role tokens; chat template registered in tokenizer_config.json.

## vLLM serving notes
- Initial support required vLLM PR #4650 to be merged (now upstream).
- SGLang also supports the model with FP8 + Torch Compile.
- Tool-call parser: vLLM has `deepseek_v3` and `deepseek_v31` parsers — neither is tuned for V2 Coder. Tool calling on V2 Coder is best-effort; treat as code-completion / instruct only unless you bring your own scaffolding.

## Quantized checkpoints
- No "official" AWQ/GPTQ from `deepseek-ai/` org; community has GGUFs and AWQs (search HF model tree). Because Lite is MoE, AWQ/GPTQ kernels for MoE were historically less mature than dense; verify vLLM supports the chosen quant + MoE combo before committing.

## Successor status
- DeepSeek-V3 / DeepSeek-V3.1 / DeepSeek-R1 etc. released subsequently. There is no direct "DeepSeek-Coder-V3-Lite" replacement at the same 16B/2.4B size — code-specialist niche has been folded into general DeepSeek-V3 line. Lite remains the most A10G-friendly DeepSeek code model.

## Release date
- Repository / paper: 2024-06-17
