---
tags: [source, models, deepseek, reasoning, mix-of-experts]
source_path: raw/deepseek-v3-r1-family.md
source_url: https://huggingface.co/deepseek-ai/DeepSeek-V3.1
ingested: 2026-05-07
last_updated: 2026-05-07
---

# DeepSeek V3.x and R1 — open-source frontier reasoning + agent

DeepSeek-V3.1 is the first open-source model to clear 60% on SWE-bench Verified
agent-mode. It and Kimi-K2 / Qwen3-Coder-480B together define the frontier of
open agentic coding. All come at the cost of needing 8× H100/H200 single-node or
multi-node deployment.

## Key claims

- **DeepSeek-V3.1**: 671B total / 37B active MoE, MIT, 128K ctx, native FP8 (UE8M0).
  - SWE-bench Verified (Agent): **66.0** (non-thinking mode)
  - LiveCodeBench (2408–2505): 56.4 / 74.8 (think)
  - Aider-Polyglot: 68.4 / 76.3 (think)
  - +20 SWE-bench points over V3-0324
- **DeepSeek-R1-0528**: 671B/37B reasoning specialist, no tool-call parser
  (R1 not function-call tuned).
- vLLM: `--tool-call-parser deepseek_v3` (mainline since 0.7.x); for R1 use
  `--reasoning-parser deepseek_r1`.
- AWS fit: needs **p5e/p5en** (8× H200 = 1.1 TB) for native FP8 with headroom,
  or p5.48xlarge (8× H100 = 640 GB) — tight at FP8. Fits comfortably on p6-b200.
  **Cannot run on any G-family instance**, even at INT4 (~340 GB).

## Pages updated on ingest

- [[models/deepseek-v3.1]] — new (multi-node flagship)
- [[infrastructure/vllm]] — `deepseek_v3` parser confirmed
- [[concepts/code-generation]] — V3.1 marks new open-source ceiling on SWE-bench
- [[comparisons/models-by-budget]] — top of "P5+ tier" recommendations
