# SWE-bench Methodology

source_url: https://www.swebench.com/SWE-bench/
fetched: 2026-05-07

## Task

"Given a codebase and an issue, a language model is tasked with generating a patch that resolves the described problem."

Concretely, an instance is a (repo, base_commit, problem_statement, gold_patch, test_patch). The model emits a unified diff. The harness applies the diff and runs the test suite. The instance is "resolved" if all PASS_TO_PASS tests still pass and FAIL_TO_PASS tests now pass.

## Variants

- **SWE-bench (full)** — original 2,294 issues from 12 popular Python repos (Django, sympy, sklearn, etc.).
- **SWE-bench Lite** — 300 instances, easier subset, designed for cheaper iteration.
- **SWE-bench Verified** (Aug 2024) — 500 issues human-validated by professional engineers from OpenAI's preparedness team. Filters out instances with ambiguous specs, missing tests, or environment issues. **Now the de-facto standard.**
- **SWE-bench Multilingual** — extended to non-Python repos.
- **SWE-bench Multimodal** (Jan 2025) — issues that require visual context (screenshots, mockups).
- **SWE-bench Pro** (2025, OpenAI) — replacement after contamination findings on Verified.

## Score

% resolved (single number, e.g. "Claude Opus 4.5 = 80.9% on SWE-bench Verified").

A score depends on (a) the model and (b) the **agent scaffold** (Aider, SWE-agent, OpenHands, custom). Leaderboard entries always note the scaffold; comparing models without the same scaffold is unreliable.

## Reproducibility

All evaluation runs in Docker. The repo provides per-instance Dockerfiles to ensure deterministic test execution.

## Contamination findings (2025)

OpenAI's late-2025 audit found frontier models (GPT-5.2, Claude Opus 4.5, Gemini 3 Flash) can reproduce verbatim gold patches or problem-statement details for some Verified instances. OpenAI now recommends SWE-bench Pro and stopped reporting Verified scores. Treat 2025/2026 numbers >75% with caution.

## What the scores mean for our wiki target models

None of the 13 wiki target models have official SWE-bench Verified scores. SWE-bench requires a competent agent scaffold, which is a separate effort from training; for sub-30B models you typically see <15% on Verified with a stock Aider/SWE-agent scaffold.
