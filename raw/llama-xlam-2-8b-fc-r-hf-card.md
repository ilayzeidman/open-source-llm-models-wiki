# Salesforce/Llama-xLAM-2-8b-fc-r — Hugging Face model card

source_url: https://huggingface.co/Salesforce/Llama-xLAM-2-8b-fc-r
fetched: 2026-05-07

## Identity

- HF model ID: `Salesforce/Llama-xLAM-2-8b-fc-r`
- Params: 8B
- Context: 128k tokens
- Base: Meta Llama 3.1 8B (hence "Llama-xLAM" prefix)
- Released: Apr 4, 2025 (paper arXiv 2504.03601)
- License: **CC-BY-NC-4.0** — research only; *also* subject to **Meta Llama 3 Community License** for the Llama-derived weights (dual-license stack)

License language quoted from card:

> "This release is for research purposes only in support of an academic paper."

## Tool-call format

OpenAI-compatible JSON. Use:

```python
inputs = tokenizer.apply_chat_template(
    messages,
    tools=tools,
    add_generation_prompt=True,
    return_dict=True,
    return_tensors="pt"
)
```

## vLLM serving

- Requires `vllm>=0.6.5`
- Uses the `xlam` tool-call parser (mainline as of newer vLLM, plugin file otherwise)

```bash
vllm serve Salesforce/Llama-xLAM-2-8b-fc-r \
  --enable-auto-tool-choice \
  --tool-parser-plugin ./xlam_tool_call_parser.py \
  --tool-call-parser xlam \
  --tensor-parallel-size 1
```

Chat template path in vLLM mainline: `examples/tool_chat_template_xlam_llama.jinja`.

## Why this checkpoint matters

This is the **8B Llama-based** xLAM-2 variant — a direct competitor to Hermes-3-Llama-3.1-8B at the same parameter count and base, but fine-tuned solely for function calling. License is more restrictive than Hermes-3 (CC-BY-NC + Llama Community vs. just Llama Community).
