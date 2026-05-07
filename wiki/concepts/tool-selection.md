---
tags: [concepts, tool-calling]
last_updated: 2026-05-07
source_count: 2
---

# Tool selection

"Tool selection" / "function calling" / "tool use" — the LLM emits a structured call (function name + JSON args) that an agent runtime executes. Quality has several distinct sub-skills.

## Sub-skills (and how BFCL measures them)

[Source: [[sources/bfcl-leaderboard-2026-05]]]

| Skill | What it tests | BFCL category |
|---|---|---|
| **Single-call accuracy** | Given the right tool exists, does the model pick it and fill args correctly? | Non-Live Single, Live Single (V1 / V2) |
| **Argument typing** | Schema types (int vs string, enum values, required fields) | AST-based parameter validation across categories |
| **Multi-call planning** | Sequence calls correctly when one depends on another | Multi-Turn Base / Augmented (V3) |
| **Parallel calls** | Emit multiple independent calls in one turn when appropriate | Parallel category (V2+) |
| **Irrelevance detection** | Refuse to call a tool when none fits, instead of forcing one | Irrelevance / Hallucination categories (V3) |
| **Multi-turn agentic** | Carry context across turns, react to tool errors, retry sensibly | Multi-Turn (V3+); τ-bench is closer to real agentic |
| **Agentic web/memory/format** | Real-world planning, format sensitivity | V4 categories (2025/2026) |

V1 is saturated. **V3 multi-turn is the current discriminator.** [Source: [[sources/bfcl-leaderboard-2026-05]]]

## Why models differ

- **Pre-training data** rarely includes much tool-call data. Most tool-calling capability comes from post-training (SFT + RLHF).
- **Specialist fine-tunes** (xLAM, Functionary, Hermes) are trained primarily on tool-use data and tend to top BFCL. They sometimes regress slightly on general capabilities.
- **Generalist models** (Llama, Qwen, Mistral, Granite) have built-in tool calling that's been progressively improved each release; they often match or beat specialists on real-world tasks.
- **Code models** (Qwen-Coder, DeepSeek-Coder, Codestral) have variable tool-calling — Qwen2.5-Coder is reportedly capable but the canonical vLLM `hermes` parser mis-extracts its output ([Source: [[sources/qwen2.5-coder-tech-report]]]).

## Format conventions in the wild

Different model families emit tool calls in different surface forms; the [[infrastructure/vllm]] parser flag handles the translation:

| Family | Format | vLLM parser |
|---|---|---|
| Llama 3.x JSON | `<\|python_tag\|>{"name": ..., "parameters": ...}` or pure JSON | `llama3_json` |
| Hermes / Qwen2.5 | `<tool_call>{"name": ..., "arguments": ...}</tool_call>` JSON-in-XML | `hermes` |
| Mistral | `[TOOL_CALLS] [{"name": ..., "arguments": ...}]` | `mistral` |
| Granite | model-specific JSON | `granite` / `granite4` |
| xLAM v2 | OpenAI-compatible JSON via `apply_chat_template(tools=...)` | `xlam` |
| DeepSeek-V3 | model-specific | `deepseek_v3` / `deepseek_v31` |
| Qwen3-Coder | XML | `qwen3_xml` |
| Pythonic | actual Python: `tool_name(arg=value)` | `pythonic` / `llama4_pythonic` |
| Functionary v3.2 | TypeScript `namespace functions { ... }` | **no mainline parser** |
| Phi-3 / Phi-4 14B | none documented | **no parser** — use constrained decoding |

[Source: [[sources/vllm-tool-calling-docs]]]

## Reliability levers

Independent of the model:

1. **Constrained decoding** (xgrammar by default in vLLM) — guarantees JSON validity; usually improves arg-correctness too.
2. **Tool schemas in the system prompt** — clearer, terser schemas → better selection.
3. **Examples in the prompt** — few-shot examples of correct tool use help small models.
4. **Smaller tool sets per turn** — give the model only the tools relevant to the current task; large catalogs hurt selection accuracy.
5. **Prefix caching** (`--enable-prefix-caching`, on by default in vLLM V1) — repeated system prompt + tool schemas reuse KV cache across turns. Big throughput win for tool-heavy workloads.

## Open questions

- **Quantization impact**: zero public benchmarks of BFCL at INT4/INT8. The Aider Ollama Q8 numbers are the only quantized tool/code numbers extant. Worth running per-model.
- **Hermes "BFCL ~91%" claim**: that figure is the well-formed-JSON rate, *not* BFCL overall accuracy. Don't quote it as comparable.
- **Qwen2.5-Coder absence from BFCL**: not on the leaderboard despite strong tool-calling claims in its tech report. Either deprecated or never submitted.

## Related
- [[concepts/benchmarks]]
- [[concepts/code-generation]]
- [[infrastructure/vllm]]
- All [[index|model pages]]

## Sources
- [[sources/bfcl-leaderboard-2026-05]]
- [[sources/vllm-tool-calling-docs]]
