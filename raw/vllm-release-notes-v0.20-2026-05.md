# vLLM Release Notes — v0.20.x (May 2026)

source_url: https://github.com/vllm-project/vllm/releases (v0.20.0, v0.20.1) and https://pypi.org/project/vllm/
fetched: 2026-05-07

## Latest stable
- **0.20.1** — May 3, 2026 (current stable)
- 0.20.0 — Apr 27, 2026
- 0.19.1 — Apr 18, 2026
- 0.19.0 — Apr 3, 2026
- 0.18.1 — Mar 31, 2026
- 0.18.0 — Mar 20, 2026
- 0.17.1 — Mar 11, 2026
- 0.17.0 — Mar 7, 2026
- 0.16.0 — Feb 26, 2026

## v0.20.0 highlights (752 commits, 320 contributors)
- PyTorch 2.11; CUDA 13.0.2 default; Python 3.14 added; **transformers >= 5 required**.
- DeepSeek V4 initial support; Hunyuan v3 preview; Granite 4.1 Vision.
- Tool/reasoning parser improvements:
  - Qwen3 treats `<tool_call>` as implicit reasoning end.
  - Mistral tool parser HF tokenizer fixes.
  - Gemma4 streaming corrections (HTML duplication, JSON corruption).
  - Reasoning parsers can access model config and expose `reasoning_start_str` / `reasoning_end_str`.
- Quantization:
  - TurboQuant 2-bit KV cache (4× capacity).
  - Per-token-head INT8/FP8 KV cache quantization.
  - NVFP4 dense models on MI300/MI355X via emulation.
  - **Online quantization frontend** unified (architectural change — `experts_int8` consolidated into FP8 online path).
  - CPU int8 compute mode in AWQ.
  - W4A16 Autoround on CPU/XPU.
  - GPTQMarlin MoE reworked (new kernel infrastructure).
  - Compressed Tensors W8A8 MXFP8 for linear/MoE.
- **Breaking**:
  - PyTorch 2.11 + CUDA 13.0.2.
  - transformers v5 baseline.
  - `vllm:prompt_tokens_recomputed` metric removed (replaced with `PrefillStats`).
  - Pooler config rename: `logit_bias`/`logit_scale` → `logit_mean`/`logit_sigma`.
  - CUDAGraph memory profiling enabled by default.
  - **Petit NVFP4 quantization removed**.
  - `Sparse24` removed.
  - `LLM.reward` deprecated → `LLM.encode`.

## v0.20.1
- "Primarily focused on DeepSeek V4 stabilization."
- Tool calling: fixed missing type conversion for non-streaming tool calls in DSV3.2/V4 (#41198).
- DeepSeek V4: PTX `cvt` instruction for faster FP32→FP4 conversion (#41015).
- Structured output: fixed reasoning-parser kwargs not being passed through to structured output (#41199).

## v0.19 highlights
- New parsers: Gemma 4 tool parser; FunctionGemma.
- Various fixes across Mistral, DeepSeek, GLM-4, OpenAI parsers.
- v0.19.1: Gemma4 streaming tool call corruption for split boolean/number values fixed.

## Pinning recommendation
- Pin `vllm==0.20.1` for current production. Avoid bumping to 0.20.0 mid-cycle (DeepSeek V4 issues fixed in 0.20.1).
- Plan around transformers v5 — older HF transformers extensions break.
- Tool-call parser names have been **additive** through 2025–2026 (no parser deletions); existing `hermes`/`llama3_json`/`mistral`/`granite`/`pythonic`/`internlm`/`jamba`/`deepseek_v3` parsers are stable and unchanged in name.
