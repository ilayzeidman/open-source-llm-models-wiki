# Devstral / Devstral-Small — Mistral AI + All Hands AI

source_urls:
  - https://huggingface.co/mistralai/Devstral-Small-2507
  - https://huggingface.co/mistralai/Devstral-Small-2-24B-Instruct-2512
  - https://huggingface.co/mistralai/Devstral-2-123B-Instruct-2512
  - https://mistral.ai/news/devstral
  - https://mistral.ai/news/devstral-2507
  - arXiv: 2509.25193
fetched: 2026-05-07

## Devstral-Small-2507 (current canonical "small")
- 24B params, dense
- Base: Mistral-Small-3.1-24B-Base-2503
- License: **Apache-2.0**
- Context: 128K (Tekken tokenizer, 131K vocab)
- SWE-bench Verified: **53.6%** (#1 open-source at release, May 2025)
- Devstral 1.0: 46.8%, Devstral Small 2: 68.0%, Devstral 2 (123B): 72.2%

## Devstral-Small-2-24B-Instruct-2512 (Dec 2025)
- 24B dense, Apache-2.0
- SWE-bench Verified: 68.0%
- Long-form agentic coding focus (OpenHands / SWE-Agent scaffolds)

## Devstral-2-123B-Instruct-2512 (flagship)
- 123B dense, Apache-2.0
- SWE-bench Verified: 72.2%
- Footprint: ~246 GB BF16 → multi-GPU; AWQ INT4 ≈ 65 GB → single 80 GB H100 fit;
  single L40S (48 GB) **does not fit at INT4** — needs g6e.12xlarge (4× L40S = 192 GB)

## vLLM serving
```
vllm serve mistralai/Devstral-Small-2507 \
  --tokenizer_mode mistral --config_format mistral --load_format mistral \
  --tool-call-parser mistral --enable-auto-tool-choice \
  --tensor-parallel-size 2
```

- Tool-call parser: `mistral` (mainline)
- Quant options: 28+ GGUF/AWQ variants on HF
- "Light enough to run on RTX 4090 / 32 GB Mac" → single A10G with AWQ INT4 (~13 GB) fits comfortably
