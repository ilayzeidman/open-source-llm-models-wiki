---
tags: [source, models, code, qwen]
source_path: raw/qwen2.5-coder-tech-report.md, raw/qwen2.5-coder-blog.md, raw/qwen2.5-coder-7b-hf-card.md, raw/qwen2.5-coder-14b-hf-card.md, raw/qwen2.5-coder-32b-hf-card.md
source_url: https://arxiv.org/abs/2409.12186
ingested: 2026-05-07
last_updated: 2026-05-07
---

# Qwen2.5-Coder family — tech report + HF cards

The Alibaba Qwen team's tech report (arXiv 2409.12186, v3 dated 2024-11-12) plus official Hugging Face model cards for the 7B / 14B / 32B Instruct variants. The single most important source for the Qwen2.5-Coder family.

## Key claims

### Family overview
- Released: **2024-11-12** (family announcement; tech report v3 same date)
- Sizes: 0.5B, 1.5B, 3B, 7B, 14B, 32B (all dense)
- Context: 32K native, 128K with YaRN (factor 4.0)
- License: **Apache-2.0** (commercial use unrestricted)
- Architecture: dense, Qwen2 family; 7B/14B/32B all use GQA (40 Q heads / 8 KV heads, varying layers)

### Per-model sizes
- 7B: 7.61B total, 6.53B non-embedding
- 14B: 14.7B total, 13.1B non-embedding (48 layers)
- 32B: 32.5B total, 31.0B non-embedding (64 layers)

### Benchmark scores from the tech report (v3, Tables 16–19)

| Benchmark | 7B-Instruct | 14B-Instruct | 32B-Instruct |
|---|---|---|---|
| HumanEval | 88.4 | 89.6 | 92.7 |
| HumanEval+ | 84.1 | 87.2 | 87.2 |
| MBPP | 83.5 | 86.2 | 90.2 |
| MBPP+ | 71.7 | 72.8 | 75.1 |
| BigCodeBench Complete | 76.9 | 81.0 | 83.0 |
| BigCodeBench Hard | 16.2 | 22.3 | 26.4 |
| BigCodeBench Instruct | 41.0 | 48.4 | 49.6 |
| LiveCodeBench (07–11/2024) | 18.2 | 23.4 | 31.4 |
| MultiPL-E Python | 87.8 | 89.0 | 92.7 |
| MultiPL-E Java | 76.5 | 79.7 | 80.4 |
| MultiPL-E C++ | 75.6 | 85.1 | 79.5 |
| MultiPL-E TS | 81.8 | 86.8 | 86.8 |
| MultiPL-E JS | 83.2 | 84.5 | 85.7 |
| CRUXEval Input-CoT | 65.8 | 69.5 | 75.2 |
| CRUXEval Output-CoT | 65.9 | 79.5 | 83.4 |
| Aider Pass@1 | 55.6 | 58.6 | 60.9 |
| Aider Pass@2 | 68.4 | 69.2 | 73.7 |

Not reported in tech report: BFCL, SWE-bench Verified.

### vLLM serving notes
- ChatML chat template; vLLM only supports static YaRN (set `rope_scaling` only when long context needed)
- Tool-call parser officially `--tool-call-parser hermes`
- ⚠ **Known parser bug**: Qwen2.5-Coder emits ```json``` fenced output instead of `<tool_call>...</tool_call>` tags, which the `hermes` parser mis-extracts. See vLLM issues [#10952](https://github.com/vllm-project/vllm/issues/10952), [#29192](https://github.com/vllm-project/vllm/issues/29192), [#32926](https://github.com/vllm-project/vllm/issues/32926). Community parser PR [#32931](https://github.com/vllm-project/vllm/pull/32931) and plugin [hanXen/vllm-qwen2.5-coder-tool-parser](https://github.com/hanXen/vllm-qwen2.5-coder-tool-parser) exist but are not merged.

### Quantized checkpoints (official, on HF)
- `Qwen/Qwen2.5-Coder-{7B,14B,32B}-Instruct-AWQ`
- `Qwen/Qwen2.5-Coder-{7B,14B,32B}-Instruct-GPTQ-Int4`
- `Qwen/Qwen2.5-Coder-{7B,14B,32B}-Instruct-GPTQ-Int8`
- The 32B-AWQ has 1.13M monthly downloads — clearly the canonical single-A10G choice.

### Successor

**Qwen3-Coder family** released February 2026. Qwen3-Coder-Next claims >70% SWE-bench Verified (massive jump from Qwen2.5-Coder-32B's unreported figure). For new deployments, evaluate Qwen3-Coder; Qwen2.5-Coder remains a strong stable baseline.

## Citation URLs
- [Tech report (arXiv 2409.12186 v3)](https://arxiv.org/html/2409.12186v3)
- [Qwen blog: qwen2.5-coder-family](https://qwenlm.github.io/blog/qwen2.5-coder-family/)
- [HF: Qwen/Qwen2.5-Coder-7B-Instruct](https://huggingface.co/Qwen/Qwen2.5-Coder-7B-Instruct)
- [HF: Qwen/Qwen2.5-Coder-14B-Instruct](https://huggingface.co/Qwen/Qwen2.5-Coder-14B-Instruct)
- [HF: Qwen/Qwen2.5-Coder-32B-Instruct](https://huggingface.co/Qwen/Qwen2.5-Coder-32B-Instruct)
- [HF: Qwen/Qwen2.5-Coder-32B-Instruct-AWQ](https://huggingface.co/Qwen/Qwen2.5-Coder-32B-Instruct-AWQ)

## Pages updated on ingest
- [[models/qwen2.5-coder-7b]], [[models/qwen2.5-coder-14b]], [[models/qwen2.5-coder-32b]]
- [[concepts/code-generation]]
- [[wiki/comparisons/tool-calling-models-on-a10g]]
