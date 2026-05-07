# vLLM Structured Outputs / Guided Decoding

source_url: https://docs.vllm.ai/en/latest/features/structured_outputs/
fetched: 2026-05-07

## Backends
- **xgrammar** — Rust-style regex; low time-per-output-token; best for long generations and reused grammars (effective caching). The default backend for tool-call format compliance in current vLLM.
- **guidance** — Rust-style regex; fast time-to-first-token even with complex grammars; output-token speed slightly slower; good for dynamic / multi-tenant workloads.
- **outlines** — Rust-style regex; older default before xgrammar.
- **lm-format-enforcer** — uses Python `re` syntax (different regex grammar from the others).

## Default
"The default backend is `auto`, which will try to choose an appropriate backend based on the details of the request."

In practice `auto` resolves to **xgrammar** for JSON-schema / grammar inputs that xgrammar can compile (which covers virtually all tool-call schemas). xgrammar became the default-fallback backend in mid-2025 (vLLM ~0.10–0.11 era) and has remained so.

## CLI flags
- Legacy: `--guided-decoding-backend {auto,xgrammar,guidance,outlines,lm-format-enforcer}`
- Current canonical form (0.18+): `--structured-outputs-config.backend=<name>`
- `--structured-outputs-config.enable_in_reasoning=True`

## Recommendation for tool calling
- Leave `auto` (= xgrammar) for sustained server throughput.
- Switch to `guidance` if you observe TTFT regression on dense schemas.
- `lm-format-enforcer` is no longer recommended for new deployments (regex dialect mismatch + lower throughput).
