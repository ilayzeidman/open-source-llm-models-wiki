# NousResearch/Hermes-4-70B — Hugging Face model card

source_url: https://huggingface.co/NousResearch/Hermes-4-70B
fetched: 2026-05-07

## Identity

- HF model ID: `NousResearch/Hermes-4-70B`
- Sister checkpoints: `NousResearch/Hermes-4-405B`, `NousResearch/Hermes-4-405B-FP8`
- Params: 70B
- Base model: Meta-Llama/Llama-3.1-70B
- License (HF tag): `llama3` (Llama 3.1 Community License)
- Released: paper dated 2025-08-25 (arXiv 2508.18255); HF card cites the same
- Context length: not explicitly restated; Llama 3.1 base = 128k

## Architecture

Hybrid-mode reasoning model. Optional `<think>...</think>` block before the final answer when "thinking" is enabled.

## Tool-calling format

Same `<tool_call>{...}</tool_call>` JSON-in-XML format as Hermes-3. The card explicitly says:

> Built-in parsers available in vLLM (`hermes`) and SGLang (`qwen25`).

System message example from the card:

```
<|start_header_id|>system<|end_header_id|>
You are a function-calling AI. Tools are provided inside <tools>…</tools>.
When appropriate, call a tool by emitting a <tool_call>{...}</tool_call> object.
After a tool responds (as <tool_response>), continue reasoning inside <think> and produce the final answer.
<tools>
{"type":"function","function":{"name":"get_weather","description":"Get weather by city","parameters":{"type":"object","properties":{"city":{"type":"string"}},"required":["city"]}}}
</tools><|eot_id|>
```

## Reasoning mode

Activate via system prompt or `thinking=True` flag. Example:

```
<|start_header_id|>assistant<|end_header_id|>
<think>
…model's internal reasoning…
</think>
Final response starts here…<|eot_id|>
```

`keep_cots=True` in chat template preserves the think block in chat history.

## Sampling defaults

`temperature=0.6, top_p=0.95, top_k=20`

## Training

Post-training corpus expanded from ~1M samples / 1.2B tokens (Hermes-3) to ~5M samples / ~60B tokens, with verified reasoning traces. No BFCL scores published on the card itself.
