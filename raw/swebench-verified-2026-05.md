# SWE-bench Verified Leaderboard (May 2026)

source_url: https://www.swebench.com/
fetched: 2026-05-07

## Status of the live page

The official SWE-bench Verified leaderboard at https://www.swebench.com/ is JavaScript-rendered. Top-50 snapshot below comes from third-party mirrors that scrape the same data.

## Snapshot from llm-stats.com (top 50 of 89)

source_url: https://llm-stats.com/benchmarks/swe-bench-verified

| Rank | Model | Org | Open/Closed | Score |
|------|-------|-----|-------------|-------|
| 1 | Claude Mythos Preview | Anthropic | Closed | 0.939 |
| 2 | Claude Opus 4.7 | Anthropic | Closed | 0.876 |
| 3 | Claude Opus 4.5 | Anthropic | Closed | 0.809 |
| 4 | Claude Opus 4.6 | Anthropic | Closed | 0.808 |
| 5 | Gemini 3.1 Pro | Google | Closed | 0.806 |
| 5 | DeepSeek-V4-Pro-Max | DeepSeek | Open | 0.806 |
| 7 | MiniMax M2.5 | MiniMax | Open | 0.802 |
| 7 | Kimi K2.6 | Moonshot AI | Open | 0.802 |
| 9 | GPT-5.2 | OpenAI | Closed | 0.800 |
| 10 | Claude Sonnet 4.6 | Anthropic | Closed | 0.796 |
| 11 | DeepSeek-V4-Flash-Max | DeepSeek | Open | 0.790 |
| 12 | Qwen3.6 Plus | Alibaba/Qwen | Closed | 0.788 |
| 13 | MiMo-V2-Pro | Xiaomi | Closed | 0.780 |
| 13 | Gemini 3 Flash | Google | Closed | 0.780 |
| 15 | GLM-5 | Zhipu AI | Open | 0.778 |
| 16 | Muse Spark | Meta | Closed | 0.774 |
| 17 | Qwen3.6-27B | Alibaba/Qwen | Open | 0.772 |
| 18 | Kimi K2.5 | Moonshot AI | Open | 0.768 |
| 19 | Seed 2.0 Pro | ByteDance | Closed | 0.765 |
| 20 | Qwen3.5-397B-A17B | Alibaba/Qwen | Open | 0.764 |
| 21 | GPT-5.1 / Instant / Thinking | OpenAI | Closed | 0.763 |
| 24 | Gemini 3 Pro | Google | Closed | 0.762 |
| 25 | GPT-5 | OpenAI | Closed | 0.749 |
| 26 | MiMo-V2-Omni | Xiaomi | Closed | 0.748 |
| 27 | Claude Opus 4.1 | Anthropic | Closed | 0.745 |
| 27 | GPT-5 Codex | OpenAI | Closed | 0.745 |
| 29 | Step-3.5-Flash | StepFun | Open | 0.744 |
| 30 | GLM-4.7 | Zhipu AI | Open | 0.738 |
| 31 | GPT-5.1 Codex | OpenAI | Closed | 0.737 |
| 32 | Seed 2.0 Lite | ByteDance | Open | 0.735 |
| 33 | Qwen3.6-35B-A3B | Alibaba/Qwen | Open | 0.734 |
| 33 | MiMo-V2-Flash | Xiaomi | Open | 0.734 |
| 35 | Claude Haiku 4.5 | Anthropic | Closed | 0.733 |
| 36 | DeepSeek-V3.2 | DeepSeek | Open | 0.731 |
| 39 | Claude Sonnet 4 | Anthropic | Closed | 0.727 |
| 40 | Claude Opus 4 | Anthropic | Closed | 0.725 |
| 41 | Qwen3.5-27B | Alibaba/Qwen | Open | 0.724 |
| 42 | Qwen3.5-122B-A10B | Alibaba/Qwen | Open | 0.720 |
| 43 | Kimi K2-Thinking-0905 | Moonshot AI | Open | 0.713 |
| 47 | Qwen3 Max | Alibaba/Qwen | Open | 0.696 |
| 47 | Qwen3-Coder-480B-A35B | Alibaba/Qwen | Open | 0.696 |
| 50 | Qwen3.5-35B-A3B | Alibaba/Qwen | Open | 0.692 |

## None of the 13 wiki target models appear on the current SWE-bench Verified leaderboard

- Qwen2.5-Coder (7B/14B/32B): no published SWE-bench Verified score from Alibaba; community evaluations exist but not on leaderboard.
- Llama-3.1-8B-Instruct, Llama-3.3-70B-Instruct: no published SWE-bench Verified score from Meta; both are reported in community studies as <10% with simple agent scaffolds (Aider/SWE-agent), but no leaderboard placement.
- DeepSeek-Coder-V2-Lite-Instruct: no SWE-bench Verified score; the larger DeepSeek-Coder-V2 (236B) is on community leaderboards.
- Mistral-Small-3 / 3.1: no Mistral self-reported SWE-bench Verified.
- Codestral-22B v0.1: no SWE-bench Verified.
- Phi-3-Medium / Phi-4: no SWE-bench Verified.
- Granite-3.1-8B / Granite-4.1-8B: no SWE-bench Verified.
- Hermes-3-Llama-3.1-8B: no SWE-bench Verified.
- xLAM-7B / xLAM-2: no SWE-bench Verified (this is a tool-calling model family, not a coding agent).
- Functionary-small: no SWE-bench Verified.

## Notes on contamination (May 2026)

OpenAI's late-2025 audit found verbatim gold-patch reproduction by frontier models on some SWE-bench Verified tasks. OpenAI now reports SWE-bench Pro instead. Treat 2025/2026 SWE-bench Verified scores >75% with caution.
