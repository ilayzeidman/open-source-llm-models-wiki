# Aider Leaderboards (May 2026)

source_url: https://aider.chat/docs/leaderboards/
fetched: 2026-05-07

Aider runs two leaderboards:

1. **Polyglot leaderboard** — 225 challenging Exercism exercises across C++, Go, Java, JavaScript, Python, Rust. The current "main" leaderboard.
2. **Code editing leaderboard (legacy)** — Python-only edit benchmark with whole/diff/architect formats. Maintained at https://aider.chat/docs/leaderboards/edit.html for older models.

YAML data file: https://github.com/Aider-AI/aider/blob/main/aider/website/_data/edit_leaderboard.yml

## Polyglot leaderboard — open-source models snapshot

| Model | Polyglot % Correct | Cost ($) | Edit Format |
|-------|--------------------|---------|-------------|
| DeepSeek-V3.2-Exp (Reasoner) | 74.2 | 1.30 | diff |
| DeepSeek R1 (0528) | 71.4 | 4.80 | diff |
| DeepSeek-V3.2-Exp (Chat) | 70.2 | 0.88 | diff |
| Qwen3-235B-A22B | 59.6 | free (self-host) | diff |
| DeepSeek R1 | 56.9 | 5.42 | diff |
| DeepSeek V3 (0324) | 55.1 | 1.12 | diff |
| Qwen3-32B | 40.0 | 0.76 | diff |
| QwQ-32B (+ Qwen 2.5) | 26.2 (architect) / 20.9 (diff) | free | architect/diff |
| DeepSeek Chat V2.5 | 17.8 | 0.51 | diff |
| Qwen2.5-Coder-32B-Instruct | 16.4 (whole) / 8.0 (diff) | free | whole/diff |
| Llama 4 Maverick | 15.6 | free | whole |
| Codestral 25.01 | 11.1 | 1.98 | whole |
| Gemma-3-27B-IT | 4.9 | free | whole |

## Edit leaderboard (legacy Python-only) — open-source models snapshot

| Model | Pass-1 | Pass-2 | Edit Format |
|-------|--------|--------|-------------|
| Qwen2.5-Coder-32B (Ollama, whole) | — | 72.9 | whole (100% well-formed) |
| Qwen2.5-Coder-14B (whole) | — | 69.2 | whole (100%) |
| Llama-3.1-405B-Instruct | 48.9 | 66.2 | whole |
| Llama-3.1-70B-Instruct | 43.6 | 58.6 | (not stated) |
| Llama-3.1-8B-Instruct | 26.3 | 37.6 | (not stated) |
| DeepSeek Coder V2 0724 | 57.9 | 72.9 | diff |
| DeepSeek V2.5 | 54.9 | 72.2 | diff |
| Qwen2 72B Instruct | 44.4 | 55.6 | whole |
| Mistral Large 2 (2407) | 39.8 | 60.2 | whole |
| Codestral-2405 | 35.3 | 51.1 | whole |
| codestral:22b-v0.1-q8_0 (Ollama) | 35.3 | 48.1 | whole |

## Per-model coverage for the 13 wiki models

| Model | On Polyglot? | On Edit (legacy)? |
|-------|--------------|-------------------|
| Qwen2.5-Coder-7B-Instruct | no | no |
| Qwen2.5-Coder-14B-Instruct | no | yes (69.2 whole, pass-2) |
| Qwen2.5-Coder-32B-Instruct | yes (16.4 whole) | yes (72.9 whole, pass-2) |
| Llama-3.1-8B-Instruct | no | yes (37.6 pass-2) |
| Llama-3.3-70B-Instruct | no | no (only 3.1-70B is on edit lb) |
| DeepSeek-Coder-V2-Lite-Instruct | no | no (only full V2) |
| Mistral-Small-3 / 3.1 | no | no |
| Codestral-22B v0.1 | yes ("Codestral 25.01" entry, 11.1) | yes (codestral-2405 35.3 / 51.1) |
| Phi-3-Medium / Phi-4 | no | no |
| Granite-3.1-8B / 4.1-8B | no | no |
| Hermes-3 | no | no |
| xLAM-7B / xLAM-2 | no | no |
| Functionary-small | no | no |

## Notes

- "Free" cost = self-hosted; uses local inference, no API call.
- Edit format dramatically affects score: Qwen2.5-Coder-32B drops from 16.4% (whole) to 8.0% (diff) on polyglot.
- Aider polyglot uses 6 languages (C++, Go, Java, JS, Python, Rust); Edit (legacy) was Python only.
