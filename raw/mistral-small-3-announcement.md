# Mistral Small 3 (announcement)

source_url: https://mistral.ai/news/mistral-small-3
fetched: 2026-05-07

## Key facts
- Release date: January 30, 2025
- Parameters: 24B
- License: Apache 2.0 (both base and instruct)
- HF ID: `mistralai/Mistral-Small-24B-Instruct-2501`
- Context: 32k (per HF card; 128k arrived in Mistral Small 3.1)

## Positioning
- Latency-optimized 24B for "the 80% of generative AI tasks"
- Mistral pitches as "open replacement for opaque proprietary models like GPT4o-mini"
- Target use cases include: low-latency function calling in agentic workflows
- Trained without RL or synthetic data (per announcement)

## Claimed performance
- Matches Llama 3.3 70B Instruct while delivering >3x faster throughput on identical hardware
- Competitive with Qwen 32B
- 81% MMLU at 150 tok/s
- "Outperforms models three times its size on code, math, and general knowledge"

## Distribution
- Mistral Le Plateforme, Hugging Face, Ollama, Kaggle, Together AI, Fireworks AI, IBM watsonx
