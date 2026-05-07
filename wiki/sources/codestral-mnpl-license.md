---
tags: [source, models, code, mistral, license]
source_path: raw/codestral-22b-hf-card.md, raw/codestral-announcement.md, raw/codestral-mnpl-license.md
source_url: https://huggingface.co/mistralai/Codestral-22B-v0.1, https://mistral.ai/licenses/MNPL-0.1.md
ingested: 2026-05-07
last_updated: 2026-05-07
---

# Codestral-22B-v0.1 + MNPL license

Mistral's 22B code specialist plus the **Mistral AI Non-Production License (MNPL-0.1)** that gates it.

## Key claims

### Codestral-22B-v0.1
- **HF model ID**: `mistralai/Codestral-22B-v0.1`
- **Params**: 22B dense
- **Context**: 32K
- **Released**: 2024-05-29
- **License**: **MNPL-0.1 — non-commercial only.** Restricted to research / testing / personal / evaluation in non-production environments. **SaaS, hosted services, and any paid-or-free commercial supply are prohibited.** Commercial license available only via paid agreement with Mistral sales.

### Benchmarks (from Mistral's announcement)
- HumanEval (Python) 86.6 (announcement) / 81.1 (community-reproduced)
- MBPP (sanitized) 91.2 / 78.2 (community)
- HumanEval-X across C++/Bash/Java/PHP/TS/C#: reported in announcement, exact values not preserved on the live page
- CruxEval, RepoBench EM, Spider: mentioned, exact values not preserved
- LiveCodeBench / SWE-bench / BFCL: NOT officially reported

### vLLM serving
- Mistral `[INST]…[/INST]` template + FIM tokens
- `--tool-call-parser mistral` exists but is tuned for Mistral-Instruct-v0.3+ and Mistral Small — Codestral was **not trained on tool-call format**; treat tool calls as unsupported.
- No first-party AWQ/GPTQ from `mistralai/` (consistent with MNPL stance); 50+ community quantizations on HF.
- Card warns: "no moderation mechanisms."

### Commercial-friendly alternatives

- **Codestral Mamba 7B** (`mistralai/Mamba-Codestral-7B-v0.1`) — Apache-2.0, Mamba2 architecture, July 2024.
- **Codestral 25.01 / Codestral V2** — Jan 2025, ~<100B params, 256K context, **NOT open-weight** (API/IDE only).
- **Devstral** family — targets later SWE-agent use cases.

Codestral-22B-v0.1 remains the only open-weight 22B dense Codestral, but its license disqualifies it from any commercial AWS deployment without a paid Mistral license.

## Citation URLs
- [HF: mistralai/Codestral-22B-v0.1](https://huggingface.co/mistralai/Codestral-22B-v0.1)
- [Mistral announcement](https://mistral.ai/news/codestral/)
- [MNPL-0.1 license text](https://mistral.ai/licenses/MNPL-0.1.md)
- [Codestral Mamba announcement](https://mistral.ai/news/codestral-mamba)

## Pages updated on ingest
- [[models/codestral-22b]]
- [[wiki/comparisons/tool-calling-models-on-a10g]]
