# Meta-Llama-3.1-8B-Instruct (Hugging Face model card)

source_url: https://huggingface.co/meta-llama/Meta-Llama-3.1-8B-Instruct
fetched: 2026-05-07

## Specifications
- Parameters: 8B (dense)
- Context: 128k tokens
- Release date: July 23, 2024
- Knowledge cutoff: December 2023
- License: Llama 3.1 Community License — https://github.com/meta-llama/llama-models/blob/main/models/llama3_1/LICENSE
- Languages: English, German, French, Italian, Portuguese, Hindi, Spanish, Thai

## License key terms (quoted from card)
> "You are granted a non-exclusive, worldwide, non-transferable and royalty-free limited license under Meta's intellectual property or other rights owned by Meta embodied in the Llama Materials..."

> "If, on the Llama 3.1 version release date, the monthly active users of the products or services made available by or for Licensee, or Licensee's affiliates, is greater than 700 million monthly active users in the preceding calendar month, you must request a license from Meta."

## Benchmarks (Instruction-Tuned, from card)
- MMLU (5-shot): 69.4
- MMLU (CoT, 0-shot): 73.0
- MMLU-Pro (CoT, 5-shot): 48.3
- HumanEval: 72.6
- MBPP++ base: 72.8
- BFCL: 76.1
- GSM-8K (CoT): 84.5
- MATH (CoT): 51.9

## Tool-call usage (transformers)
- `tokenizer.apply_chat_template(messages, tools=[get_current_temperature], add_generation_prompt=True)`
- assistant tool_calls schema: `[{"type": "function", "function": {"name": ..., "arguments": {...}}}]`
- tool result role: `{"role": "tool", "name": ..., "content": ...}`

## Training
- Pretraining tokens: ~15T
- Compute: 1.46M H100-80GB GPU hours
- Fine-tuning data: public + 25M+ synthetic examples
