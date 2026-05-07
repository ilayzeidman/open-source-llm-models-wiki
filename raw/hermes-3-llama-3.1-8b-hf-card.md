# NousResearch/Hermes-3-Llama-3.1-8B — Hugging Face model card

source_url: https://huggingface.co/NousResearch/Hermes-3-Llama-3.1-8B
fetched: 2026-05-07

## Identity

- HF model ID: `NousResearch/Hermes-3-Llama-3.1-8B`
- Params: 8B
- Base model: `meta-llama/Llama-3.1-8B`
- Tensor type: BF16
- License (HF tag): `llama3` — i.e. inherits the **Llama 3.1 Community License**
- Tech report: https://arxiv.org/abs/2408.11857 (Hermes 3 Technical Report, Teknium / Quesnelle / Guang, Aug 2024)
- Function-calling repo: https://github.com/NousResearch/Hermes-Function-Calling
- Released: Aug 15, 2024 (arxiv submission)
- Context length: 128k (Llama 3.1 base; not explicitly restated on card)

## Open LLM Leaderboard scores (from card)

- Average: 23.49
- IFEval (0-shot): 61.70
- BBH (3-shot): 30.72
- MATH Lvl 5 (4-shot): 4.76
- GPQA (0-shot): 6.38
- MuSR (0-shot): 13.62
- MMLU-PRO (5-shot): 23.77

(BFCL not reported on the card.)

## Tool-calling format

System prompt structure (ChatML-style headers, JSON inside `<tool_call>` blocks):

```
<|im_start|>system
You are a function calling AI model. You are provided with function signatures within <tools></tools> XML tags. You may call one or more functions to assist with the user query. Don't make assumptions about what values to plug into functions. Here are the available tools: <tools> {function_definitions} </tools>
Use the following pydantic model json schema for each tool call you will make: {"properties": {"arguments": {"title": "Arguments", "type": "object"}, "name": {"title": "Name", "type": "string"}}, "required": ["arguments", "name"], "title": "FunctionCall", "type": "object"}
For each function call return a json object with function name and arguments within <tool_call></tool_call> XML tags as follows:
<tool_call>
{"arguments": <args-dict>, "name": <function-name>}
</tool_call><|im_end|>
```

Assistant emits:

```
<|im_start|>assistant
<tool_call>
{"arguments": {"symbol": "TSLA"}, "name": "get_stock_fundamentals"}
</tool_call><|im_end|>
```

This is the canonical "Hermes" format that vLLM's `--tool-call-parser hermes` is built for.

## Chat template

ChatML with `<|im_start|>` / `<|im_end|>` markers. Use `tokenizer.apply_chat_template(..., add_generation_prompt=True)`.
