# EvalPlus Leaderboard (May 2026)

source_url: https://evalplus.github.io/leaderboard.html
fetched: 2026-05-07

## Status

The official EvalPlus leaderboard is JavaScript-rendered and could not be fetched in tabular form. Per-model HumanEval / HumanEval+ / MBPP / MBPP+ scores below are collected from primary sources (model cards, papers, official blogs).

EvalPlus is the "Plus" series: it extends HumanEval (164 problems) and MBPP (~378 problems sanitized) with **additional tests** that catch more incorrect solutions. EvalPlus reports both the original (HumanEval / MBPP) and the augmented (HumanEval+ / MBPP+) pass@1 numbers, all 0-shot, greedy decoding, deterministic.

## Per-model scores from primary sources

### Qwen2.5-Coder Instruct family
Source: Qwen2.5-Coder Technical Report (arxiv 2409.12186 v3).

| Model | HumanEval | HE+ | MBPP | MBPP+ |
|-------|-----------|-----|------|-------|
| Qwen2.5-Coder-7B-Instruct | 88.4 | 84.1 | 83.5 | 71.7 |
| Qwen2.5-Coder-14B-Instruct | 89.6 | 87.2 | 86.2 | 72.8 |
| Qwen2.5-Coder-32B-Instruct | 92.7 | 87.2 | 90.2 | 75.1 |

(For context, Base models from same report: 7B 61.6/53.0/76.9/62.9, 14B 64.0/57.9/81.0/66.7, 32B 65.9/60.4/83.0/68.2.)

### Llama 3.1 / 3.3 family
Source: meta-llama/llama-models eval_details (cited via community).

| Model | HumanEval (0-shot) | MBPP EvalPlus |
|-------|--------------------|---------------|
| Llama-3.1-8B-Instruct | 72.6 | 72.8 |
| Llama-3.3-70B-Instruct | 88.4 | 87.6 |

### DeepSeek-Coder-V2
Source: DeepSeek-Coder-V2 paper (arxiv 2406.11931).

| Model | HumanEval | MBPP+ | LCB |
|-------|-----------|-------|-----|
| DeepSeek-Coder-V2-Lite-Instruct (16B/2.4B) | 81.1 | 68.8 | 24.3 |
| DeepSeek-Coder-V2-Instruct (236B/21B) | 90.2 | 76.2 | 43.4 |

### Codestral-22B v0.1
Source: Mistral announcement (https://mistral.ai/news/codestral).

| Model | HumanEval | MBPP |
|-------|-----------|------|
| Codestral-22B-v0.1 | 86.6 | 91.2 |

### Mistral Small 3 / 3.1 / 3.2 (24B)
Source: Mistral announcements + community.

| Model | HumanEval | HE+ | MBPP / MBPP+ |
|-------|-----------|-----|--------------|
| Mistral-Small-3 (24B, 2501) | not officially published as HE/MBPP; "competitive with Llama 3.3 70B" claim only |  |  |
| Mistral-Small-3.1 (24B, 2503) | 88.4 | — | 74.7 (MBPP) |
| Mistral-Small-3.2 (24B, 2506) | — | 88.99–92.90 | MBPP pass@5 74.63–78.33 |

### Phi family (Microsoft)
- Phi-3-Medium-128k-Instruct: HumanEval ~62.2 (Microsoft tech report), MBPP ~75.0 *(unverified — needs source)*.
- Phi-4 (14B, dense): HumanEval ~82.6, MBPP ~71.5 *(unverified — needs source ascription)*. Microsoft evaluated Phi-4 on LiveCodeBench 2024-08 to 2025-01.

### Granite (IBM)
- Granite-3.1-8B-Instruct model card does NOT publish HumanEval / MBPP. It reports OpenLLM Leaderboard V1/V2 only (MMLU 65.34, IFEval 72.08, etc.).
- Granite-4.1-8B-Instruct: HumanEval 87.2, MBPP 82–87, BFCL v3 68.27, IFEval 87.06, GSM8K 92.49 (IBM blog).

### Hermes-3-Llama-3.1-8B
- HumanEval: not published in model card (model card lists MMLU, GPQA, BBH, IFEval). Coding likely tracks base Llama-3.1-8B (~72.6 HE).

### xLAM family (Salesforce)
- xLAM models are tool-calling action models. HumanEval / MBPP / LCB not part of their evaluation suite.

### Functionary (MeetKai)
- Same: function-calling specialist; coding benchmarks not published.

## Quantization disclosure

All numbers above are FP16 / BF16 unless explicitly noted. Aider entries marked "Ollama" use Q8_0 GGUF and are noted in the Aider raw file.
