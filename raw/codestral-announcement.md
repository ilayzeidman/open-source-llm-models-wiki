# Codestral — Mistral AI announcement

source_url: https://mistral.ai/news/codestral/
fetched: 2026-05-07

## Release
- Released: 2024-05-29
- Model: Codestral-22B-v0.1
- Vendor: Mistral AI

## Headline specs
- 22B parameters
- 32K context window (vs 4K/8K/16K for then-current peers)
- 80+ programming languages: Python, Java, C, C++, JavaScript, Bash, Swift, Fortran included
- Two modes: Instruct + Fill-in-the-Middle (FIM)

## License
- Mistral AI Non-Production License (MNPL-0.1) — research / testing only
- Commercial license required for production; obtained by contacting Mistral sales
- Free `codestral.mistral.ai` IDE endpoint during 8-week beta (waitlist)
- Token-billed `api.mistral.ai` endpoint for third-party app developers

## Benchmark claims (from announcement; specific numbers were given in the post but Mistral does not always preserve exact tables in the live page)
- HumanEval pass@1 (Python): 81.1 [community-reproduced]
- MBPP sanitised pass@1: 78.2 [community-reproduced]
- CruxEval: results shown, exact numbers not in fetched excerpt
- RepoBench EM: Codestral wins on long context
- Spider (SQL): used; exact score not in excerpt
- HumanEval-X across C++, Bash, Java, PHP, TypeScript, C#: reported
- FIM compared favorably vs DeepSeek-Coder-33B

## Successor mentions in the announcement
- None — successors (Codestral Mamba, Codestral 25.01) postdate this announcement.
