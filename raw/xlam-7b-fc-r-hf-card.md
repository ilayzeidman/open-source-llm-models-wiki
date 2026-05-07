# Salesforce/xLAM-7b-fc-r — Hugging Face model card

source_url: https://huggingface.co/Salesforce/xLAM-7b-fc-r
fetched: 2026-05-07

## Identity

- HF model ID: `Salesforce/xLAM-7b-fc-r`
- Params: 6.91B
- Context: **4k tokens** (note: significantly shorter than xLAM-7b-r at 32k)
- Released: Jul 17, 2024
- License: **CC-BY-NC-4.0** — non-commercial; ALSO subject to the [DeepSeek model license](https://github.com/deepseek-ai/deepseek-coder/blob/main/LICENSE-MODEL) (this variant is built on DeepSeek-Coder, not Llama)
- Tensor type: BF16

## "fc" vs "r" naming

- `xLAM-7b-fc-r`: smaller (6.91B), 4k ctx, **fine-tuned specifically for function calling**, based on DeepSeek-Coder-7B. The "fc" series is the specialist family.
- `xLAM-7b-r`: 7.24B, 32k ctx, broader agent/tool-use general model (different base).

The card explicitly directs general-purpose users away: "For more specialized function calling models, please take a look into our `fc` series."

## BFCL standing on card

- BFCL benchmark cutoff 07/18/2024
- Overall accuracy: **88.24%**
- Rank: 3rd at the time
- BFCL **v1** (multi-turn / v3 / v4 not yet existing in July 2024)

## Tool-call output format

JSON list of calls under a `tool_calls` key:

```json
{
    "tool_calls": [
        {"name": "func_name1", "arguments": {"argument1": "value1", "argument2": "value2"}},
        {"name": "func_name2", "arguments": {"argument3": "value3"}}
    ]
}
```

Example:
- query: "What's the weather like in New York in fahrenheit?"
- response: `{"tool_calls": [{"name": "get_weather", "arguments": {"location": "New York", "unit": "fahrenheit"}}]}`

## Required prompt scaffold

```
[BEGIN OF TASK INSTRUCTION]
{task_instruction}
[END OF TASK INSTRUCTION]

[BEGIN OF AVAILABLE TOOLS]
{tools_json}
[END OF AVAILABLE TOOLS]

[BEGIN OF FORMAT INSTRUCTION]
{format_instruction}
[END OF FORMAT INSTRUCTION]

[BEGIN OF QUERY]
{query}
[END OF QUERY]
```

## Successor

xLAM-2 family released Apr 2025 (`Salesforce/xLAM-2-32b-fc-r`, `Salesforce/Llama-xLAM-2-8b-fc-r`, `Salesforce/xLAM-2-3b-fc-r`, `Salesforce/xLAM-2-1b-fc-r`) — supersedes the v1 fc-r checkpoints. Same CC-BY-NC-4.0 license though.
