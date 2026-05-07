# Log

Append-only chronological record. Format: `## [YYYY-MM-DD] <op> | <subject>` where op ∈ {bootstrap, ingest, query, lint, refactor, note}.

## [2026-05-07] bootstrap | scaffold + seed initial research pages

- Created schema files: [[CLAUDE.md]] (pointer), [[AGENTS.md]] (source of truth).
- Created [[index.md]] catalog and this log.
- Seeded `wiki/` with overview, hardware (A10G + multi-GPU), infrastructure (vLLM, NVIDIA Dynamo, quantization), concepts (tool selection, code generation, benchmarks), 13 model pages, and the master comparison table at [[comparisons/tool-calling-models-on-a10g]].
- All numbers in seeded pages were marked `*(unverified — needs source)*`.

## [2026-05-07] ingest | first research sweep — 56 primary sources, 10 source pages

Ran a parallel multi-agent web research sweep covering hardware/AWS pricing, vLLM, NVIDIA Dynamo, all major benchmarks (BFCL, SWE-bench Verified, LiveCodeBench, Aider, EvalPlus), and HF model cards / tech reports / vendor blogs for every wiki candidate model.

### Raw sources saved (56 files in `raw/`)
Hardware: aws-g5-instance-types, aws-g5-pricing, aws-p4-instance-types, aws-p5-instance-types, nvidia-a10-datasheet, nvidia-a10g-vs-a10, nvidia-dynamo-readme.
vLLM: vllm-tool-calling-docs (×2), vllm-tool-call-parsers (×2), vllm-quantization-docs, vllm-quantization-ampere, vllm-engine-args, vllm-release-notes-v0.20, vllm-structured-outputs.
Benchmarks: bfcl-leaderboard, bfcl-methodology, bfcl-overview, swebench-verified, swebench-methodology, livecodebench (×2), aider-leaderboard, evalplus-leaderboard.
Models: qwen2.5-coder-{7b,14b,32b}-hf-card + tech-report + blog; deepseek-coder-v2-lite-hf-card + paper; codestral-22b-hf-card + announcement + mnpl-license; llama-3.1-8b-hf-card + community-license; llama-3.3-70b-hf-card; mistral-small-{2501,3.1,3.2}-hf-card + 3 + 3.1 announcements; phi-3-medium-128k-hf-card + phi-4-hf-card; granite-{3.0-announcement,3.1,3.3}-cards; hermes-3-8b-hf-card + hermes-4-70b-hf-card; xlam-{7b-r,7b-fc-r,2-8b-fc-r,2-32b-fc-r}-hf-card + cc-by-nc-license; functionary-readme.

### Source pages created (10, in `wiki/sources/`)
- [[sources/aws-ec2-pricing-2026-05]]
- [[sources/nvidia-a10g-specs]]
- [[sources/nvidia-dynamo-readme]]
- [[sources/vllm-tool-calling-docs]]
- [[sources/vllm-quantization-docs]]
- [[sources/bfcl-leaderboard-2026-05]]
- [[sources/code-benchmarks-2026-05]]
- [[sources/qwen2.5-coder-tech-report]]
- [[sources/deepseek-coder-v2-paper]]
- [[sources/codestral-mnpl-license]]
- [[sources/llama-3.x-cards]]
- [[sources/mistral-small-3.x-cards]]
- [[sources/phi-3-and-phi-4-cards]]
- [[sources/granite-3.x-cards]]
- [[sources/hermes-3-and-4-cards]]
- [[sources/xlam-family-cards]]
- [[sources/functionary-readme]]

### Wiki pages updated
All 22 wiki pages were updated with verified numbers and `[Source: [[sources/...]]]` citations. Replaced every `*(unverified — needs source)*` marker. Updated `last_updated` and `source_count` frontmatter on each.

### Material corrections from the seed
- **A10G FP16 dense compute**: ~70 TF (not 125; that's the data-center A10). AWS's "250 TOPS" claim for G5 appears to reference the A10.
- **Mistral-Small canonical version is 3.2 (2506)** with 128K context and improved tool-calling template — supersedes both 3.1 and 2501.
- **Granite 3.3 supersedes 3.1** — HumanEval went from "not on card" to 89.73; reasoning tags added.
- **DeepSeek-Coder-V2-Lite license IS commercial-OK** — initial seed left this uncertain. DeepSeek License explicitly permits commercial use.
- **All Salesforce xLAM checkpoints (v1 + v2, every size) are CC-BY-NC-4.0** — non-commercial. No exceptions in the family.
- **Codestral-22B v0.1's MNPL is firmly non-commercial.** Codestral Mamba 7B (Apache-2.0) is the commercial alternative.
- **Phi-3-medium and Phi-4 14B both lack vLLM tool-call parsers** — agentic disqualified despite strong reasoning numbers.
- **Functionary needs MeetKai's `server_vllm.py` fork** — no mainline vLLM parser as of 0.20.1.
- **NVIDIA Dynamo's own README**: "If you're running a single model on a single GPU, your inference engine alone is probably sufficient." Single-A10G value is essentially nil.
- **vLLM tool-call parsers expanded substantially** since the seed: added `xlam`, `deepseek_v3`/`v31`, `qwen3_xml`, `granite4`, `llama4_pythonic`, `openai`, `kimi_k2`, plus ~10 more model-family-specific parsers. Names purely additive across 2025–2026.
- **Qwen2.5-Coder is absent from BFCL** despite tool-calling claims; vLLM `hermes` parser is known-flaky on the Coder variant (issues #10952, #29192, #32926).

### Pages touched (28 wiki + 17 source = 45)
- All 13 model pages
- 2 hardware pages (a10g-g5xlarge, multi-gpu-options)
- 3 infrastructure pages (vllm, nvidia-dynamo, quantization)
- 3 concept pages (tool-selection, code-generation, benchmarks)
- 1 master comparison
- 1 overview
- 1 index
- 17 source pages (10 standalone + 7 model-family consolidated)
