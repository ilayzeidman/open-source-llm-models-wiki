---
tags: [concepts, tool-calling]
last_updated: 2026-05-07
source_count: 0
---

# Tool selection

"Tool selection" / "function calling" / "tool use" — the LLM emits a structured call (function name + JSON args) that an agent runtime executes. Quality has several distinct sub-skills.

## Sub-skills

| Skill | What it tests |
|---|---|
| **Single-call accuracy** | Given the right tool exists, does the model pick it and fill args correctly? |
| **Argument typing** | Does it respect schema types (int vs string, enum values, required fields)? |
| **Multi-call planning** | Does it sequence calls correctly when one depends on another? |
| **Parallel calls** | Does it emit multiple independent calls in one turn when appropriate? |
| **Irrelevance detection** | Does it *refuse* to call a tool when none fits, instead of forcing an inappropriate one? |
| **Multi-turn agentic** | Does it carry context across turns, react to tool errors, and retry sensibly? |

[[concepts/benchmarks]] enumerates which benchmarks measure which sub-skill.

## Why models differ

- **Pre-training data** rarely includes much tool-call data. Most tool-calling capability comes from post-training (SFT + RLHF).
- **Specialist fine-tunes** (xLAM, Functionary, Hermes) are trained primarily on tool-use data and tend to top BFCL. They sometimes regress slightly on general capabilities.
- **Generalist models** (Llama, Qwen, Mistral, Granite) have built-in tool calling that's been progressively improved each release; they hit a ceiling slightly below specialists on BFCL but cover wider use cases.
- **Code models** (Qwen-Coder, DeepSeek-Coder, Codestral) often have strong tool calling because code-style structured output is similar territory.

## Format conventions in the wild

Different model families emit tool calls in different surface forms; the [[infrastructure/vllm]] parser flag handles the translation:

- **Llama 3.x JSON** — `<|python_tag|>{"name": ..., "parameters": ...}` or pure JSON.
- **Hermes** — XML-ish `<tool_call>{...}</tool_call>` blocks.
- **Mistral** — `[TOOL_CALLS] [{"name": ..., "arguments": ...}]`.
- **Pythonic** — emits actual Python: `tool_name(arg=value)`.
- **OpenAI-compatible JSON** — most fine-tunes converge on this.

## Reliability levers

Independent of the model:

1. **Constrained decoding** (xgrammar / outlines) — guarantees JSON validity; usually improves arg-correctness too.
2. **Tool schemas in the system prompt** — clearer, terser schemas → better selection.
3. **Examples in the prompt** — few-shot examples of correct tool use help small models.
4. **Smaller tool sets per turn** — give the model only the tools relevant to the current task; large catalogs hurt selection accuracy.

## Related
- [[concepts/benchmarks]]
- [[concepts/code-generation]]
- [[infrastructure/vllm]]
- All [[index|model pages]]

## Sources
- (none yet)

## TODO / verify
- BFCL leaderboard categories and what each measures
- Recent papers comparing specialist vs generalist tool-calling models
- vLLM constrained-decoding impact on BFCL
