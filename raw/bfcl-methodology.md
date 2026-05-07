# BFCL Methodology

source_urls:
- https://gorilla.cs.berkeley.edu/blogs/8_berkeley_function_calling_leaderboard.html (V1)
- https://gorilla.cs.berkeley.edu/blogs/13_bfcl_v3_multi_turn.html (V3)
- https://github.com/ShishirPatil/gorilla/blob/main/berkeley-function-call-leaderboard/README.md
fetched: 2026-05-07

## Versions

- **V1** (Feb 2024): Single-turn AST + executable function calling. ~2,000 Q-F-A pairs.
- **V2** (Aug 2024): Added live (real-world) data, weighted to reduce evaluator bias.
- **V3** (Sep 2024): Added multi-turn and multi-step categories. ~1,000 new entries.
- **V4** (2026): Added agentic capabilities — web search, memory management, format-sensitivity testing.

## V1 categories

### Python
- **Simple Function**: 1 invocation of 1 documented function. (400 entries)
- **Multiple Function**: 1 invocation chosen from 2–4 functions. (200)
- **Parallel Function**: Multiple concurrent invocations of 1 function from a single prompt. (200)
- **Parallel Multiple**: Combines parallel + multiple. (200)

### Non-Python
- **REST API**: GET/POST with path/query params. (70 executable)
- **Java**: Type-aware AST. (100 AST)
- **JavaScript**: Type-aware AST. (50 AST)

### Special
- **Function Relevance Detection** (Irrelevance): Model must withhold a call when no function fits. (240 entries)
- **Chatting Capability**: General Q&A without invocation.

### Evaluation
- **AST Evaluation**: parse the model's call → match function name → match required params → typecheck. Type rules: bool exact, list/tuple ordered match, string case-insensitive whitespace-normalized, dict by key+value.
- **Executable Evaluation**: run the call. Non-REST: exact / real-time (±20%) / structural match. REST: response type + JSON key validation.

## V3 additions

- **Multi-Turn Base** (200): straight multi-turn dialog with function output feeding next call.
- **Multi-Turn Augmented** (800): missing parameters / missing functions / long-context / composite scenarios.
- **Hallucination Measurement** (240+): model must not call non-existent functions.

### Scoring
- **State-based** (write/delete ops): compare backend state after execution to expected state.
- **Response-based** (read ops): subset-match against minimal viable execution paths.

## V3 leaderboard composition

| Category | Entries | Aggregation |
|----------|---------|-------------|
| Non-Live (Single-Turn) | 1,390 | unweighted average |
| Live (Single-Turn) | 2,251 | weighted average |
| Multi-Turn Base | 200 | unweighted |
| Multi-Turn Augmented | 800 | unweighted |
| Hallucination | 240+ | — |
| Irrelevance | 240 | — |

## Saturation status

- V1 saturated quickly (top models >90%).
- V2 still has headroom (~85% for top).
- V3 multi-turn is the discriminator (top GLM-4.5 at 0.778 overall, multi-turn typically <60%).

## Contamination concerns

- Live data is real-world API/function specs that may overlap with training data.
- Multi-turn is largely synthetic (Persona Hub generated, expert reviewed) — lower contamination risk.

## Output formats supported

- "Function Calling" (FC): native tool-call schema.
- "Prompt": prompt-engineered, parsed from text.
A given model can appear under both labels with different scores; the FC variant typically wins.

## Test categories file

`TEST_CATEGORIES.md` in the gorilla repo lists current categories. Examples mentioned: `simple_python`, `parallel`, `live_multiple`, `multi_turn`, `multi_turn_base`, `web_search` (V4).

## Supported models (subset relevant to wiki)

From SUPPORTED_MODELS.md in the gorilla repo (2026-04-12):

- **Qwen**: Qwen3 family (0.6B–32B), Qwen3-235B-A22B-Instruct-2507, QwQ-32B. (Note: Qwen2.5-Coder explicitly absent from current supported list.)
- **Llama**: Llama-3.1-8B/70B (FC + Prompt), Llama-3.2-1B/3B (FC), Llama-3.3-70B (FC), Llama-4 Maverick/Scout.
- **DeepSeek**: DeepSeek-V3.2-Exp (FC + Prompt), DeepSeek-R1 (Prompt, self-hosted).
- **Mistral**: mistral-large-2411, Mistral-Medium-2505, Mistral-Small-2506, Open-Mistral-Nemo-2407.
- **Phi**: Phi-4 (Prompt only), Phi-4-mini (FC + Prompt).
- **Granite**: Multiple versions support FC.
- **xLAM-2**: All variants 1B–70B (FC enabled).
- **Functionary**: Medium and Small v3.1 (FC).

The 💻 marker indicates self-hosted via vLLM/sglang.
