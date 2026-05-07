# Codestral-22B-v0.1 — Hugging Face model card

source_url: https://huggingface.co/mistralai/Codestral-22B-v0.1
fetched: 2026-05-07

## Identity
- HF model ID: `mistralai/Codestral-22B-v0.1`
- Vendor: Mistral AI
- Announcement: https://mistral.ai/news/codestral/
- License URL: https://mistral.ai/licenses/MNPL-0.1.md

## Architecture / parameters
- Total parameters: 22B
- Tensor type: BF16
- Context length: 32K (per Mistral announcement; not visible on HF card)
- Capabilities: Instruct + Fill-in-the-Middle (FIM)

## License — Mistral AI Non-Production License (MNPL-0.1)
- Free use ONLY for "testing, research, Personal, or evaluation purposes in Non-Production Environments"
- Explicitly disallows commercial activity: "shall not supply the Mistral Models or Derivatives in the course of a commercial activity, whether in return for payment or free of charge" (covers SaaS, hosted services, cloud, etc.)
- Distribution must include the agreement and the attribution notice "Licensed by Mistral AI under the Mistral AI Non-Production License"
- Modifications allowed; cannot misrepresent as official Mistral product
- Termination clauses for breach or IP litigation
- Commercial license available on request from Mistral sales — separate paid agreement

## Programming languages
- Trained on 80+ programming languages (Python, Java, C, C++, JavaScript, Bash, Swift, Fortran, etc.)

## Benchmarks (per Mistral announcement / community measurements)
- HumanEval (Python pass@1): 81.1
- MBPP (sanitised pass@1): 78.2
- CruxEval, RepoBench EM, Spider (SQL): mentioned but exact numbers not in the announcement page itself
- HumanEval-X across C++, Bash, Java, PHP, TypeScript, C#: reported, exact numbers not on announcement
- CodeLlama-70B HE pass@1 cited as 67.1; DeepSeek-Coder-33B 77.4; Llama-3-70B 76.2 — Codestral 22B beats all three on HE.
- LiveCodeBench / BFCL / SWE-bench: NOT reported officially.

## Chat template
- Mistral instruction template (`[INST]...[/INST]` style) registered in tokenizer_config.json
- FIM: prefix/suffix/middle tokens for code-completion mode

## Tool calling
- Not advertised. Codestral predates Mistral's tool-calling-aware models. vLLM `--tool-call-parser mistral` exists but is tuned for Mistral-Instruct-v0.3+/Mistral-Large; using it on Codestral is unsupported.

## vLLM serving
```bash
vllm serve mistralai/Codestral-22B-v0.1
# add --tool-call-parser mistral --chat-template examples/tool_chat_template_mistral_parallel.jinja
# only if you've validated tool calls work for your prompt set
```

## Quantized checkpoints
- 50 community quantizations on HF model tree (no first-party AWQ/GPTQ from mistralai org — likely intentional given MNPL).

## Release / successor status
- Released 2024-05-29
- Successors:
  - Codestral Mamba (`mistralai/Mamba-Codestral-7B-v0.1`) — Apache-2.0, Mamba2 architecture, July 2024
  - Codestral 25.01 (a.k.a. Codestral V2) — January 2025, ~<100B parameters, 256K context, NOT open-weight (API/IDE only)
  - Devstral models (later) target SWE-agent use cases
- Codestral-22B-v0.1 is the only open-weight Codestral with a 22B dense backbone, but it is NOT free to deploy commercially without a paid Mistral license.

## Key limitation note from card
> "The Codestral-22B-v0.1 does not have any moderation mechanisms."
