# vLLM tool-call parser support matrix

source_url: https://docs.vllm.ai/en/latest/features/tool_calling/
fetched: 2026-05-07

## Complete mainline parser list (vLLM 2026)

| Parser | Models | CLI |
|--------|--------|-----|
| `hermes` | NousResearch Hermes 2 Pro / Hermes 3 / Hermes 4 (Llama-3 backed). Also works for Qwen2.5 models whose tokenizer ships Hermes-style template | `--tool-call-parser hermes` |
| `mistral` | Mistral-7B-Instruct-v0.3 etc. | `--tool-call-parser mistral` |
| `llama3_json` | Llama 3.1, 3.2 | `--tool-call-parser llama3_json --chat-template examples/tool_chat_template_llama3.1_json.jinja` |
| `llama4_pythonic` | Llama 4 | `--tool-call-parser llama4_pythonic --chat-template examples/tool_chat_template_llama4_pythonic.jinja` |
| `granite` | IBM Granite 3.0/3.1 | `--tool-call-parser granite --chat-template examples/tool_chat_template_granite.jinja` |
| `granite4` | IBM Granite 4.0 | `--tool-call-parser granite4` |
| `internlm` | InternLM2.5-7B-Chat | `--tool-call-parser internlm --chat-template examples/tool_chat_template_internlm2_tool.jinja` |
| `jamba` | AI21 Jamba-1.5 | `--tool-call-parser jamba` |
| **`xlam`** | **Salesforce Llama-xLAM-2-\*-fc-r and Qwen-xLAM-\*** | `--tool-call-parser xlam --chat-template examples/tool_chat_template_xlam_llama.jinja` (or `xlam_qwen.jinja`) |
| `deepseek_v3` | DeepSeek-V3-0324, R1-0528 | `--tool-call-parser deepseek_v3 --chat-template examples/tool_chat_template_deepseekv3.jinja` |
| `deepseek_v31` | DeepSeek-V3.1 | `--tool-call-parser deepseek_v31 --chat-template examples/tool_chat_template_deepseekv31.jinja` |
| `openai` | GPT-OSS-20B/120B | `--tool-call-parser openai` |
| `kimi_k2` | Moonshot Kimi-K2-Instruct | `--tool-call-parser kimi_k2` |
| `pythonic` | Llama 3.2 (pythonic), ToolACE-8B, Ultravox, Llama 4 Scout/Maverick | `--tool-call-parser pythonic --chat-template examples/tool_chat_template_llama3.2_pythonic.jinja` |
| `functiongemma` | FunctionGemma-270M-IT | `--tool-call-parser functiongemma --chat-template examples/tool_chat_template_functiongemma.jinja` |
| `qwen3_xml` | Qwen3-Coder-480B / 30B-A*B-Instruct | `--tool-call-parser qwen3_xml` |
| `olmo3` | AllenAI Olmo-3-7B/32B-Think | `--tool-call-parser olmo3` |
| `gigachat3` | GigaChat3 | `--tool-call-parser gigachat3` |

## Notes salient to tool-calling specialists

- **Hermes 2 Theta** is flagged as having "degraded tool call quality" — avoid.
- **Functionary**: NO mainline parser exists. Use MeetKai's `server_vllm.py` or run via SGLang.
- **xLAM v1 (`xLAM-7b-r`, `xLAM-7b-fc-r`)**: not in the parser table — the `xlam` parser is officially scoped to the v2 `Llama-xLAM-2-*-fc-r` and `Qwen-xLAM-*` checkpoints. v1 outputs may need the plugin script `xlam_tool_call_parser.py` from the HF repo.
- Custom user parsers can be registered via `--tool-parser-plugin`.
