---
tags: [source, infrastructure, vllm]
source_path: raw/vllm-tool-calling-docs-2026-05.md, raw/vllm-tool-call-parsers.md, raw/vllm-tool-calling-parsers.md, raw/vllm-tool-calling-docs.md
source_url: https://docs.vllm.ai/en/latest/features/tool_calling/
ingested: 2026-05-07
last_updated: 2026-05-07
---

# vLLM tool-calling docs (2026-05, vLLM 0.20.x)

The current authoritative parser list and per-model recommendations as of vLLM 0.20.1.

## Key claims

### CLI flags (verbatim from docs)
- `--enable-auto-tool-choice`
- `--tool-call-parser <name>`
- `--chat-template <path>`
- `--tool-parser-plugin <path>` — for out-of-tree custom parsers (used by Functionary, custom Qwen2.5-Coder parser)

### Parser table (vLLM 0.20.x)

| Parser | Target families | Chat template needed? |
|---|---|---|
| `hermes` | Nous Hermes-2 Pro / 2 Theta / 3 / 4; Qwen2.5/Qwen3 base instruct | No (uses tokenizer's) |
| `mistral` | Mistral-7B-Instruct-v0.3, Mistral-Small (3.x), Codestral, Devstral | `tool_chat_template_mistral_parallel.jinja` |
| `llama3_json` | Llama 3.1 / 3.2 / 3.3 | `tool_chat_template_llama3.1_json.jinja` |
| `pythonic` | Llama-3.2 (pythonic), ToolACE, Ultravox | matching `_pythonic.jinja` |
| `llama4_pythonic` | Llama 4 | `tool_chat_template_llama4_pythonic.jinja` |
| `granite` | Granite 3.0, 3.1, 3.3 | `tool_chat_template_granite.jinja` (3.0); 3.1+ ship template in tokenizer |
| `granite4` | Granite 4.0 / 4.1 | — |
| `granite-20b-fc` | Granite-20B-FunctionCalling | `tool_chat_template_granite_20b_fc.jinja` |
| `internlm` | InternLM2.5-7B-Chat | — |
| `jamba` | AI21 Jamba-1.5 | — |
| `xlam` | Salesforce/Qwen xLAM (v2 family) | `tool_chat_template_xlam_{llama,qwen}.jinja` |
| `deepseek_v3` | DeepSeek-V3, DeepSeek-R1 | `tool_chat_template_deepseekv3.jinja` |
| `deepseek_v31` | DeepSeek-V3.1 | — |
| `qwen3_xml` | Qwen3-Coder | — |
| `openai` | GPT-OSS | — |
| `kimi_k2` | Kimi-K2-Instruct | — |
| `hunyuan_a13b` | Hunyuan-A13B-Instruct | — |
| `cohere_command3` | Command-A-Reasoning-08-2025 | — |
| `longcat`, `glm45`, `glm47`, `functiongemma`, `olmo3`, `gigachat3`, `minimax` | model-family specific | — |

Parsers added across 2025–2026 are purely additive — no renames or deletions. Pages can safely cite `hermes` / `llama3_json` / `mistral` / `granite` / `pythonic` / `xlam` / `deepseek_v3` as stable.

### Per-model recommendations (the 13 wiki candidates)

| Model | Recommended parser | Status |
|---|---|---|
| Qwen2.5-Coder (any size) | `hermes` (officially) | **Known-flaky**: model emits ```json``` fences, not `<tool_call>` tags. Issues #10952, #29192, #32926. Community parser PR #32931 unmerged. Alt: hanXen/vllm-qwen2.5-coder-tool-parser plugin. |
| Llama 3.1 / 3.3 | `llama3_json` + `tool_chat_template_llama3.1_json.jinja` | Official. No parallel tool calls (Llama 3.x limitation). |
| Mistral-Small 3.x / Codestral | `mistral` + `tool_chat_template_mistral_parallel.jinja` | Official. Mistral-Small 3.2 ships an improved parallel-call template. |
| DeepSeek-Coder-V2-Lite | `deepseek_v3` (no V2-specific parser) | **Unverified for V2** — `deepseek_v3` is V3-tuned; V2-Coder may misbehave. |
| Granite 3.1 / 3.3 | `granite` (3.1+) ; `granite4` for 4.x | Official |
| Hermes-3 | `hermes` | Official; canonical target for the `hermes` parser. |
| Salesforce xLAM v2 | `xlam` + matching `_llama` or `_qwen` template | Official since vLLM 0.6.5 |
| MeetKai Functionary | **No mainline parser.** | Use MeetKai's `server_vllm.py` fork or `--tool-parser-plugin` route. Confirmed missing from vLLM 0.20.1 parser list. |
| Phi-3 / Phi-4 | **No official parser.** | Community workaround = `llama3_json` + custom template; brittle. Open issues: #14682 (Phi-4-mini), #15788. Phi-4-mini (3.8B) added native function calling but base Phi-4 14B did not. |

## Pages updated on ingest
- [[infrastructure/vllm]]
- [[concepts/tool-selection]]
- All [[wiki/comparisons/tool-calling-models-on-a10g|model pages]]
