# Berkeley Function-Calling Leaderboard (BFCL) — version history & 2026 status

source_url: https://gorilla.cs.berkeley.edu/leaderboard.html
fetched: 2026-05-07

## Versions

- **BFCL v1** (Feb 2024) — single-turn AST-evaluated function calls. Used by xLAM v1 release announcements (07/18/2024 cutoff).
- **BFCL v2** (Aug 2024) — added enterprise-/community-contributed real-world functions. Used by xLAM-7b-r release (09/03/2024 cutoff) and Functionary v3.x announcements.
- **BFCL v3** (late 2024) — introduced multi-turn interactions. Cited by xLAM-2 paper (APIGen-MT) and τ-bench-style evaluation.
- **BFCL v4** (current as of Apr 2026) — adds holistic agentic evaluation (memory, web search, format sensitivity).

The v4 leaderboard (last updated 2026-04-12) is the canonical "current" reference.

## Current top tier (V4 snapshot, May 2026)

Reported in search summaries; the live leaderboard dynamically reorders:

- Llama 3.1 405B Instruct: 88.5%
- Llama 3.1 70B Instruct: 84.8%
- Llama 3.1 8B Instruct: 76.1%
- GLM-4.5 (Zhipu) leads BFCL v3 at 0.778

## Caveat

Explicit BFCL v4 entries for `xLAM-2-32b-fc-r`, `Hermes-4-70B`, and `functionary-medium-v3.2` were not extractable from the leaderboard HTML in this fetch. The model cards and papers cite older BFCL versions:

| Model | Self-cited BFCL | Version |
|-------|-----------------|---------|
| xLAM-7b-fc-r | 88.24% (rank 3) | v1 (07/2024) |
| xLAM-7b-r | (image only) | v2 (09/2024) |
| xLAM-2-* | image charts in paper | v3 |
| functionary-medium-v3.1 | 88.88% (rank 2) | v2 |
| functionary-small-v3.2 | 82.82% | v2 |
| functionary-small-v3.1 | 82.53% | v2 |
| Hermes-3-Llama-3.1-8B | not reported on card | — |
| Hermes-4-70B | not reported on card | — |

When citing scores in the wiki, **always** annotate the BFCL version — v2 vs v3 vs v4 are not directly comparable.
