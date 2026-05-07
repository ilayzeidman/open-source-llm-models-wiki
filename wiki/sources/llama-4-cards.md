---
tags: [source, models, meta, llama, multimodal]
source_path: raw/llama-4-cards.md
source_url: https://www.llama.com/models/llama-4/
ingested: 2026-05-07
last_updated: 2026-05-07
---

# Llama 4 — Scout and Maverick

Meta's first natively multimodal MoE family. Notable for the **10M-token context** on
Scout and the 17B-active-parameter design that matches frontier efficiency.

## Key claims

- **Llama-4-Scout-17B-16E-Instruct**: 109B total / 17B active (16 experts).
  Llama 4 Community License. **10M ctx**. INT4 ≈ 55 GB → fits single H100 (80 GB)
  or **8× L40S g6e.48xlarge** with TP=8 INT4 (~7 GB/GPU).
- **Llama-4-Maverick-17B-128E-Instruct**: ~400B total / 17B active (128 experts).
  Multi-node only on AWS open-source paths.
- Tool calling: documented prompt format; community reports note inconsistent JSON
  adherence vs DeepSeek/Qwen3 — recommend output validation/retry in production.
- Benchmarks (Scout / Maverick): MMLU Pro 74.3 / 80.5, GPQA Diamond 57.2 / 69.8,
  LiveCodeBench 32.8 / 43.4. Both trail Qwen3-Coder/DeepSeek-V3.1 on coding.
- License is non-OSI: **commercial use OK for orgs <700M MAU**; not Apache/MIT.

## Pages updated on ingest

- [[models/llama-4-scout]] — new
- [[concepts/tool-selection]] — note Llama 4 reliability caveat
- [[comparisons/models-by-budget]] — Scout listed in g6e.48xl / p4d / p5 tiers
