# Salesforce/xLAM-2-32b-fc-r — Hugging Face model card

source_url: https://huggingface.co/Salesforce/xLAM-2-32b-fc-r
fetched: 2026-05-07

## Identity

- HF model ID: `Salesforce/xLAM-2-32b-fc-r`
- Params: 33B
- Context: 32k default, up to 128k via YaRN
- Base model: Qwen 2.5-32B
- Released: Apr 4, 2025 (paper arXiv 2504.03601 — APIGen-MT)
- License: **CC-BY-NC-4.0** — research only

License language quoted from card:

> "This release is for research purposes only in support of an academic paper."
> "not specifically designed or evaluated for all downstream purposes"

## xLAM-2 family

| HF ID | Params | Base | Context |
|-------|--------|------|---------|
| `Salesforce/xLAM-2-1b-fc-r` | 1B | Qwen 2.5-1.5B | 32k |
| `Salesforce/xLAM-2-3b-fc-r` | 3B | Qwen 2.5-3B | 32k |
| `Salesforce/Llama-xLAM-2-8b-fc-r` | 8B | Llama 3.1 8B | 128k |
| `Salesforce/xLAM-2-32b-fc-r` | 33B | Qwen 2.5-32B | 32k (128k YaRN) |
| (`xLAM-2-70b-fc-r` cited in paper) | 70B | Llama 3.1 70B | 128k |

## Benchmarks

- Trained with APIGen-MT data
- Evaluated on **BFCL v3** and **τ-bench**
- xLAM-2-70b-fc-r: τ-bench overall success rate **56.2%** (pass@1)
  - Beats Llama 3.1 70B Instruct (38.2%), DeepSeek V3 (40.6%), GPT-4o (52.9%)
  - Approaches Claude 3.5 Sonnet (60.1%)
- BFCL v3 overall scores rendered as image on card; not extractable as text

## Tool-call format / vLLM serving

OpenAI-compatible function calling. **Requires custom tool-parser plugin** for vLLM:

```bash
wget https://huggingface.co/Salesforce/xLAM-2-1b-fc-r/raw/main/xlam_tool_call_parser.py

vllm serve Salesforce/xLAM-2-32b-fc-r \
  --enable-auto-tool-choice \
  --tool-parser-plugin ./xlam_tool_call_parser.py \
  --tool-call-parser xlam
```

NOTE: As of vLLM ≥ 0.7 the `xlam` parser ships **mainline** (per vLLM tool_calling docs), so the `--tool-parser-plugin` step is no longer always required. Chat template: `examples/tool_chat_template_xlam_qwen.jinja` (Qwen base) or `examples/tool_chat_template_xlam_llama.jinja` (Llama-xLAM-2).

## Related papers

- APIGen-MT: arXiv 2504.03601 (Apr 2025) — multi-turn data generation pipeline
- ActionStudio: arXiv 2503.22673 (Mar 2025) — training framework
