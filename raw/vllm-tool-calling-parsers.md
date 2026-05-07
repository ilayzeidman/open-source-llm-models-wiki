# vLLM Tool-Calling Parsers — reference

source_url: https://docs.vllm.ai/en/latest/features/tool_calling/
fetched: 2026-05-07

## Supported `--tool-call-parser` values (subset relevant to wiki)
- `hermes` — Nous Hermes / Qwen2.5 family (Qwen3 too)
- `mistral` — Mistral-Instruct v0.3+ / Mistral-Large
- `llama3_json` — Llama 3.1 / 3.2 JSON
- `llama4_pythonic` — Llama 4 pythonic
- `granite`, `granite4`, `granite-20b-fc` — IBM Granite
- `internlm`, `jamba`, `xlam`, `pythonic`
- `deepseek_v3`, `deepseek_v31` — DeepSeek-V3 family
- `minimax`, `functiongemma`, `qwen3_xml`, `olmo3`, `gigachat3`, others

## Recommendations relevant to the wiki

### Qwen2.5 / Qwen2.5-Coder
- Documented: `--enable-auto-tool-choice --tool-call-parser hermes` (Qwen2.5 chat template includes Hermes-style tool support)
- Reality for **Qwen2.5-Coder** specifically:
  - GitHub vllm-project/vllm issues #10952, #29192, #32926 report that Qwen2.5-Coder does NOT reliably emit `<tool_call>` tags; it produces fenced ```json``` or raw JSON instead.
  - Community workaround: dedicated `<tools>`-tag parser at `hanXen/vllm-qwen2.5-coder-tool-parser`, being upstreamed via PR #32931.
  - Until upstreamed, treat Qwen2.5-Coder as not-production-ready for native vLLM tool calling.

### Qwen3 / Qwen3-Coder
- `--enable-auto-tool-choice --tool-call-parser hermes` — works as documented.

### DeepSeek-Coder-V2 (Lite or full)
- No dedicated parser. `deepseek_v3` / `deepseek_v31` are tuned for V3-era models, not V2 Coder.
- Tool-calling on V2 Coder is unsupported; use prompt-engineered ReAct or schema-prompting only.

### Codestral-22B-v0.1
- `--tool-call-parser mistral` exists but targets Mistral-Instruct/Large. Codestral is a code-completion / FIM model and was not trained on Mistral's tool-call format. Treat tool calls as unsupported.
