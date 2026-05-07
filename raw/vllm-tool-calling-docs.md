# vLLM tool-calling parser reference

source_url: https://docs.vllm.ai/en/latest/features/tool_calling/
mirror: https://github.com/vllm-project/vllm/blob/main/docs/features/tool_calling.md
fetched: 2026-05-07

## Parser → model mapping (CLI flags)

### `llama3_json`
- Models: `meta-llama/Llama-3.1-*`, `meta-llama/Llama-3.2-*`, `meta-llama/Llama-4-*` (also works for Llama 3.3 in practice)
- Flags: `--tool-call-parser llama3_json --chat-template <template>`
- Templates: `examples/tool_chat_template_llama3.1_json.jinja`, `examples/tool_chat_template_llama3.2_json.jinja`
- Caveat: "Parallel tool calls are not supported for Llama 3" (only Llama 4)

### `mistral`
- Original supported model: `mistralai/Mistral-7B-Instruct-v0.3` (and used in practice for Mistral Small 3.x)
- Flags (HF format): `--tokenizer_mode hf --config_format hf --load_format hf --tool-call-parser mistral --chat-template examples/tool_chat_template_mistral_parallel.jinja`
- Flags (Mistral format, recommended for Mistral Small 3.x): `--tokenizer_mode mistral --config_format mistral --load_format mistral --tool-call-parser mistral --enable-auto-tool-choice`
- Templates: `examples/tool_chat_template_mistral.jinja`, `examples/tool_chat_template_mistral_parallel.jinja`
- Caveat: "Mistral 7B struggles to generate parallel tool calls correctly" (Small 3.2 explicitly improves the template)

### `hermes`
- Models: NousResearch Hermes-2-Pro, Hermes-2-Theta, Hermes-3
- Flags: `--tool-call-parser hermes`
- Caveat: "Hermes 2 Theta models are known to have degraded tool call quality"

### `granite` / `granite4` / `granite-20b-fc`
- `--tool-call-parser granite` — Granite 3.1-8b (no chat-template override needed)
- `--tool-call-parser granite --chat-template examples/tool_chat_template_granite.jinja` — Granite 3.0-8b
- `--tool-call-parser granite4` — Granite 4.0
- `--tool-call-parser granite-20b-fc --chat-template examples/tool_chat_template_granite_20b_fc.jinja` — Granite 20b-fc

### `pythonic`
- Models: `meta-llama/Llama-3.2-1B-Instruct`, Team-ACE/ToolACE-8B, `meta-llama/Llama-4-Scout-17B-16E-Instruct`
- Flags: `--tool-call-parser pythonic --chat-template <template>`
- Caveat: "The model must not generate both text and tool calls in the same generation"

### `llama4_pythonic`
- Models: `meta-llama/Llama-4-*`
- Flags: `--tool-call-parser llama4_pythonic --chat-template examples/tool_chat_template_llama4_pythonic.jinja`

### Phi-4
- No `phi4_*` parser exists in vLLM as of mid-2025.
- Base Phi-4 14B: no native tool support.
- Phi-4-mini function calling support was tracked in vllm-project/vllm#14682.
