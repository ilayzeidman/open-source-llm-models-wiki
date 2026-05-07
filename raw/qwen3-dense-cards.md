# Qwen3 dense models — HF cards

source_urls:
  - https://huggingface.co/Qwen/Qwen3-32B
  - https://huggingface.co/Qwen/Qwen3-14B
  - https://huggingface.co/Qwen/Qwen3-8B
  - https://qwenlm.github.io/blog/qwen3/
  - https://arxiv.org/pdf/2505.09388
fetched: 2026-05-07

## Qwen3-32B (dense)
- 32.8B total / 31.2B non-embedding
- License: Apache-2.0
- Architecture: 64 layers, 64 Q heads / 8 KV heads (GQA)
- Context: 32,768 native, 131,072 with YaRN
- Hybrid reasoning: thinking / non-thinking mode toggle (`enable_thinking`,
  or `/think` / `/no_think` soft tags)
- Sampling (thinking mode): T=0.6, top_p=0.95, top_k=20 (no greedy decoding)
- vLLM serve: `--enable-reasoning --reasoning-parser deepseek_r1`
  (the deepseek_r1 reasoning parser handles `<think>...</think>` blocks)
- Quantized variants: 150+ on HF
- Footprint: ~62 GB BF16; AWQ INT4 ≈ 18 GB — single A10G / L4 fit (tight ctx) or
  single L40S (comfortable)

## Qwen3-14B (dense)
- 14B total params, Apache-2.0
- Same hybrid reasoning architecture
- Context 32K native (131K YaRN)
- AWQ INT4 ≈ 9 GB → single A10G with comfortable ctx

## Qwen3-8B (dense)
- 8B total, Apache-2.0
- Hybrid reasoning, same parser flags
- AWQ INT4 ≈ 5 GB → cheapest g4dn / g6.xlarge fit

## Qwen3 BFCL v3 (technical report)
- Qwen3-235B-A22B: 70.8 BFCL v3
- Smaller dense variants score in the 60s on BFCL v3
- All Qwen3 models use FC (function-calling) format for BFCL eval, with yarn for 64K
  multi-turn

## Tool calling parser
- vLLM mainline: `--tool-call-parser hermes` works for Qwen3 dense (used as the
  generic parser for Qwen3 chat template).
- For Qwen3-Coder specifically, use `qwen3_coder` parser instead.
