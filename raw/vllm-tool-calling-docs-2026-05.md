# vLLM Tool Calling Documentation

source_url: https://docs.vllm.ai/en/latest/features/tool_calling/
fetched: 2026-05-07

## CLI Flags

Three key flags enable tool calling:

- `--enable-auto-tool-choice` (mandatory): "Auto tool choice. It tells vLLM that you want to enable the model to generate its own tool calls"
- `--tool-call-parser`: Specifies which parser to use
- `--chat-template` (optional): Path to template handling tool-role messages
- `--tool-parser-plugin`: register custom out-of-tree parsers (Python file implementing `ToolParser` with `extract_tool_calls()` and `extract_tool_calls_streaming()`)

## Complete Parser List with Supported Models (as of vLLM 0.20.x, May 2026)

| Parser Name | Target Models |
|---|---|
| `hermes` | Nous Research Hermes-2 Pro, Hermes-2 Theta, Hermes-3 (also recommended for Qwen2.5/Qwen3 base instruct models — chat template in their `tokenizer_config.json` is Hermes-style) |
| `mistral` | Mistral-7B-Instruct-v0.3 and compatible Mistral models (Mistral-Small, Codestral/Devstral). Format: `[TOOL_CALLS]name{...args...}` |
| `llama3_json` | Llama 3.1, Llama 3.2 (JSON-based tool calling) |
| `llama4_pythonic` | Llama 4 models |
| `pythonic` | Llama-3.2 (pythonic mode), ToolACE, Ultravox, Llama-4 Scout/Maverick |
| `granite` | IBM Granite 3.0, 3.1 |
| `granite4` | IBM Granite 4.0 |
| `granite-20b-fc` | IBM Granite-20b-functioncalling |
| `internlm` | InternLM2.5-7b-chat |
| `jamba` | AI21 Jamba-1.5 models |
| `xlam` | Salesforce / Qwen xLAM models (both Llama-based and Qwen-based variants) |
| `minimax` | MiniMax-M1 models |
| `deepseek_v3` | DeepSeek-V3, DeepSeek-R1 |
| `deepseek_v31` | DeepSeek-V3.1 |
| `openai` | GPT-OSS models |
| `kimi_k2` | Kimi-K2-Instruct |
| `hunyuan_a13b` | Hunyuan-A13B-Instruct |
| `cohere_command3` | Command-A-Reasoning-08-2025 |
| `longcat` | LongCat-Flash-Chat models |
| `glm45` | GLM-4.5, GLM-4.6 |
| `glm47` | GLM-4.7 |
| `functiongemma` | Google FunctionGemma-270m-it |
| `qwen3_xml` | Qwen3-Coder models |
| `olmo3` | Allenai Olmo-3 |
| `gigachat3` | GigaChat-3 |

## Per-model recommendations & example commands

### Llama 3.1 / 3.2 (JSON)
```
vllm serve meta-llama/Llama-3.1-8B-Instruct \
    --enable-auto-tool-choice \
    --tool-call-parser llama3_json \
    --chat-template examples/tool_chat_template_llama3.1_json.jinja
```
For Llama 3.2 use `tool_chat_template_llama3.2_json.jinja` (or `_pythonic` with `--tool-call-parser pythonic`).

### Llama 4
`--tool-call-parser llama4_pythonic --chat-template examples/tool_chat_template_llama4_pythonic.jinja`

### Mistral / Codestral / Devstral
For parallel tool calls:
`--tokenizer_mode hf --config_format hf --load_format hf --tool-call-parser mistral --chat-template examples/tool_chat_template_mistral_parallel.jinja`

### DeepSeek-V3 / R1
`--tool-call-parser deepseek_v3 --chat-template examples/tool_chat_template_deepseekv3.jinja`
DeepSeek-V3.1: use `deepseek_v31`.

### IBM Granite
- Granite 3.0/3.1: `--tool-call-parser granite --chat-template examples/tool_chat_template_granite.jinja`
- Granite 4.0: `--tool-call-parser granite4`
- Granite 20B FC: `--tool-call-parser granite-20b-fc --chat-template examples/tool_chat_template_granite_20b_fc.jinja`

### Hermes / Qwen2.5 / Qwen3
`--tool-call-parser hermes` (no template flag — chat templates from tokenizer_config.json are sufficient).

### Salesforce xLAM
- Llama-based: `--tool-call-parser xlam --chat-template examples/tool_chat_template_xlam_llama.jinja`
- Qwen-based: `--tool-call-parser xlam --chat-template examples/tool_chat_template_xlam_qwen.jinja`

### Qwen2.5-Coder / Qwen3-Coder
- Qwen2.5-Coder is **not** a clean fit for `hermes`: its training format emits ```json``` code-fenced calls, not `<tool_call>` XML tags. Use a community parser plugin (e.g. hanXen/vllm-qwen2.5-coder-tool-parser) or accept that built-in parsers will mis-extract some calls.
- Qwen3-Coder: `--tool-call-parser qwen3_xml`.

### Functionary (MeetKai)
Mainline vLLM does **not** ship a built-in functionary parser. MeetKai hosts a vLLM-based fork/companion (`MeetKai/functionary` repo, `server_vllm.py`) with custom raw-response parsing logic. To run Functionary on stock vLLM you must register a custom `--tool-parser-plugin`.

### Phi-3 / Phi-4
No dedicated parser shipped. Workarounds in the wild use `--tool-call-parser llama3_json` together with a custom chat template, but this is not officially documented and is brittle (community issue #14682, #15788).

## Parameters
- `parallel_tool_calls=false` ensures vLLM only returns zero or one tool call per request.
