# Log

Append-only chronological record. Format: `## [YYYY-MM-DD] <op> | <subject>` where op ∈ {bootstrap, ingest, query, lint, refactor, note}.

## [2026-05-07] bootstrap | scaffold + seed initial research pages

- Created schema files: [[CLAUDE.md]] (pointer), [[AGENTS.md]] (source of truth).
- Created [[index.md]] catalog and this log.
- Seeded `wiki/` with overview, hardware (A10G + multi-GPU), infrastructure (vLLM, NVIDIA Dynamo, quantization), concepts (tool selection, code generation, benchmarks), 13 model pages, and the master comparison table at [[comparisons/tool-calling-models-on-a10g]].
- All numbers in seeded pages are marked `*(unverified — needs source)*` where they come from prior knowledge rather than an ingested source. First ingest priorities are listed in each page's `## TODO / verify` section.
