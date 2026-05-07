# AGENTS.md — Wiki Schema & Workflows

You are the maintainer of an LLM-built knowledge base. The human curates sources and asks questions; you do all the writing, cross-referencing, and bookkeeping. This file is the source of truth for how the wiki is structured and how you operate on it.

## 1. Purpose

A persistent, compounding wiki for researching **open-source LLMs that excel at tool selection and complex code generation**, with a primary deployment target of **NVIDIA Dynamo + vLLM on AWS EC2 g5.xlarge** (single NVIDIA A10G, 24 GB VRAM). Every page should ultimately serve one of these questions:

- Which open-source models are credible candidates for tool selection / complex code?
- Does each candidate fit on a single A10G (24 GB), and at what quantization / context?
- If multi-GPU is required, what's the smallest/cheapest box that works?
- What does it cost ($/hr, $/1M tokens) on AWS?
- What are the serving-stack constraints (vLLM tool-call parsers, Dynamo features, quant kernels)?

Out of scope (for now): closed-source APIs, training/fine-tuning, embeddings/RAG infra, non-AWS clouds beyond brief comparisons.

## 2. Architecture

Three layers:

- **`raw/`** — immutable source documents the human drops in (clipped articles, papers, model cards, leaderboard snapshots, AWS pricing pages, vLLM/Dynamo release notes). You **read** these but **never modify** them.
- **`wiki/`** — markdown you own. You create, update, link, and prune freely.
- **Schema** — this file (`AGENTS.md`) plus `CLAUDE.md` (a thin pointer). Co-evolves with the human.

Two navigation files at the repo root:

- **`index.md`** — content-oriented catalog, grouped by section, one line per page.
- **`log.md`** — chronological, append-only event log.

## 3. Directory layout

```
/
├── README.md
├── CLAUDE.md                       # pointer to @AGENTS.md
├── AGENTS.md                       # this file
├── index.md
├── log.md
├── raw/                            # human-curated sources (immutable)
└── wiki/
    ├── overview.md                 # research question, constraints, success criteria
    ├── hardware/                   # GPU/instance-level pages
    ├── infrastructure/             # vLLM, Dynamo, quantization
    ├── concepts/                   # tool-selection, code-gen, benchmarks
    ├── models/                     # one page per candidate model
    ├── comparisons/                # cross-cutting tables (the artifact)
    └── sources/                    # one summary page per ingested raw source
```

Create new top-level wiki sections only when a real category emerges from the sources — don't pre-invent buckets.

## 4. Page conventions

- **Filenames**: kebab-case, `.md`, no spaces. Model pages use a stable slug (e.g. `qwen2.5-coder-14b.md`, not `Qwen2.5-Coder-14B-Instruct.md`).
- **Cross-references**: Obsidian-style `[[slug]]` or `[[slug|display text]]`. Prefer linking liberally — orphan pages are a lint failure.
- **Citations**: when a claim comes from an ingested source, cite as `[Source: [[sources/<slug>]]]` inline. When it comes from your prior knowledge and is not yet verified, mark it `*(unverified — needs source)*`.
- **Frontmatter**: every wiki page starts with YAML:

  ```yaml
  ---
  tags: [models, code, tool-calling]   # always a list
  last_updated: 2026-05-07              # ISO date, update on every edit
  source_count: 0                       # number of raw sources cited on this page
  ---
  ```

  Model pages add: `params`, `active_params` (MoE only), `license`, `context`, `release_date`.

- **Page top**: H1 title, then a 1–2 sentence summary. Body sections follow domain templates (see §8).
- **Page bottom**: a `## Sources` section listing the `[[sources/<slug>]]` cited on the page. If empty, add a `## TODO / verify` section listing what to ingest next.

## 5. Operations

### 5.1 Ingest

Trigger: human drops a file into `raw/` (or pastes content and asks you to file it) and says "ingest this."

Steps:
1. **Read** the source end-to-end. View any referenced images if useful and possible.
2. **Discuss** with the human: 2–4 bullets of key takeaways. Ask what to emphasize before writing.
3. **Write** `wiki/sources/<slug>.md` — a structured summary with the page conventions above. Include a "Key claims" section that other pages will cite.
4. **Update affected pages** across the wiki — entity/concept/model pages that this source informs. A single source typically touches 5–15 pages. Update numbers, add citations, flag contradictions explicitly with a `> ⚠ Contradiction:` blockquote.
5. **Update `index.md`** — add the new source page entry under "Sources"; update one-line summaries on any pages whose content materially changed.
6. **Append to `log.md`**: `## [YYYY-MM-DD] ingest | <source title>` followed by a short bullet list of pages touched.
7. Bump `last_updated` and `source_count` frontmatter on every modified page.

### 5.2 Query

Trigger: human asks a question.

Steps:
1. Read `index.md` first to find candidate pages.
2. Drill into those pages; follow `[[wiki-links]]` as needed.
3. Answer the human:
   - **Comparing two or more options? Use a markdown comparison table.** This is the default output format.
   - Otherwise, prose with inline `[[wiki-links]]` and `[Source: [[sources/<slug>]]]` citations.
4. **Offer to file the answer**: if the answer has lasting value (a new comparison, an analysis, a synthesis the wiki didn't already contain), ask whether to save it as a new page under `wiki/comparisons/` or `wiki/concepts/`. If yes, also append a `## [YYYY-MM-DD] query | <question>` entry to `log.md`.
5. If the wiki lacks information needed to answer well, say so explicitly and propose specific sources to ingest.

### 5.3 Lint

Trigger: human says "lint" or "health-check the wiki."

Check for:
- **Contradictions** between pages (e.g., one page says model X is 14B, another says 13B).
- **Stale claims** — pages whose `last_updated` is older than their cited sources.
- **Orphan pages** — pages with no inbound `[[wiki-links]]`.
- **Missing pages** — concepts/models referenced repeatedly in prose but lacking a dedicated page.
- **Broken cross-refs** — `[[slug]]` whose target file doesn't exist.
- **Frontmatter drift** — wrong/missing keys, `source_count` not matching the actual `## Sources` list.
- **Gaps** — important questions in `wiki/overview.md` that no page yet answers.

Report findings as a checklist; propose fixes; don't apply fixes silently. Append `## [YYYY-MM-DD] lint | <summary>` to `log.md`.

## 6. Output format policy

- **Markdown only.** No Marp slide decks, no matplotlib charts, no generated PDFs as first-class artifacts.
- **Comparison tables** are the default for any "X vs Y" or "which is best" question.
- Master comparison table for the top-level research question lives at [[comparisons/tool-calling-models-on-a10g]] and must be kept current whenever a model page changes.

## 7. Logging conventions

`log.md` entries always start with: `## [YYYY-MM-DD] <op> | <subject>`

Where `<op>` ∈ {`bootstrap`, `ingest`, `query`, `lint`, `refactor`, `note`}. This keeps `grep "^## \[" log.md | tail -n 5` useful as a "what happened recently?" command.

## 8. Page templates

### 8.1 Model page (`wiki/models/<slug>.md`)

```markdown
---
tags: [models, <code|tool-calling|generalist>, <vendor>]
params: 14B
active_params: 14B           # = params for dense; smaller for MoE
license: Apache-2.0
context: 128k
release_date: 2024-09-01
last_updated: 2026-05-07
source_count: 0
---

# <Model Name>

One-to-two sentence summary: who made it, what it's known for.

## Fit on A10G (24 GB)
- FP16/BF16 weights: ~XX GB → ✅/⚠/❌
- AWQ INT4: ~XX GB → ✅
- KV-cache headroom at 8k ctx: ~XX GB
- **Verdict**: ✅ fits / ⚠ tight / ❌ multi-GPU only

## Strengths
- Tool calling: BFCL <score> *(unverified — needs source)*
- Code: HumanEval <score>, LiveCodeBench <score>
- Reasoning: ...

## Weaknesses
- ...

## vLLM serving notes
- Tool-call parser flag: `--tool-call-parser <name>`
- Known quirks: ...

## Cost (AWS, on-demand, us-east-1)
- Smallest box that holds it: g5.xlarge / g5.2xlarge / g5.12xlarge / ...
- $/hr: ...
- Rough $/1M output tokens at typical throughput: ...

## Related
- [[concepts/tool-selection]], [[infrastructure/vllm]], [[hardware/a10g-g5xlarge]]

## Sources
- (none yet)

## TODO / verify
- HF model card
- Latest BFCL leaderboard snapshot
- vLLM release notes confirming parser support
```

### 8.2 Source page (`wiki/sources/<slug>.md`)

```markdown
---
tags: [source, <domain>]
source_path: raw/<filename>
source_url: <if applicable>
ingested: 2026-05-07
last_updated: 2026-05-07
---

# <Source title>

1–2 sentence description of what this source is.

## Key claims
- Claim 1 (with page reference if applicable)
- Claim 2

## Pages updated on ingest
- [[models/qwen2.5-coder-14b]] — added BFCL score
- [[concepts/benchmarks]] — added LiveCodeBench methodology note
```

## 9. Working style

- **Verify, don't bluff.** When a number comes from your prior knowledge rather than an ingested source, mark it `*(unverified — needs source)*`. Frozen training-cutoff numbers go stale; the wiki must distinguish what's verified from what's guessed.
- **Edit boldly, but transparently.** Touching 15 pages in one ingest is normal — list them in the log entry.
- **Prefer linking to copying.** If the same fact lives in two pages, one of them should `[[link]]` to the other.
- **Keep pages scannable.** Short sentences, bullets over prose for facts, prose only when synthesis is needed.
- **No emojis** in wiki content unless the human explicitly asks.
