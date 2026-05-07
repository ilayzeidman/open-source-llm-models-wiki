# vLLM Engine / Serve Arguments

source_url: https://docs.vllm.ai/en/latest/configuration/engine_args.html and /configuration/conserving_memory/
fetched: 2026-05-07

## Memory & serving flags (current names, vLLM 0.20.x)

| Flag | Default | Description |
|---|---|---|
| `--gpu-memory-utilization` | `0.92` (was 0.9 prior to 0.18) | Fraction of GPU memory used by model executor + KV cache. 0..1. |
| `--max-model-len` | auto from model config | Total context length (prompt+output). Accepts `1k`, `25.6k`, `auto`, `-1`. |
| `--max-num-seqs` | engine-default (256 historically); now varies per scheduler | Max concurrent sequences in a batch iteration. |
| `--swap-space` | 4 (GiB per GPU) | CPU swap space for KV-cache offload. |
| `--enforce-eager` | `False` | Disables CUDA graph capture; reduces VRAM at cost of throughput. |
| `--enable-prefix-caching` | on by default in V1 (since 0.16) | Reuses KV cache across requests sharing a prefix. |
| `--enable-chunked-prefill` | on by default in V1 | Splits prefill across iterations. |
| `--tensor-parallel-size` | `1` | TP group size. Must equal number of GPUs in a single replica. |
| `--quantization` | None (autodetected from `quantization_config`) | Force a specific method: awq, awq_marlin, gptq, gptq_marlin, fp8, bitsandbytes, gguf, ... |
| `--kv-cache-dtype` | `auto` | `auto`, `fp8`, `fp8_e5m2`, `fp8_e4m3`, `int8` |
| `--dtype` | `auto` | `auto`, `bfloat16`, `float16`, `float32` |

## Tool-calling flags
- `--enable-auto-tool-choice`
- `--tool-call-parser <name>`
- `--chat-template <path>`
- `--tool-parser-plugin <python_file>`

## Structured-outputs flags (renamed in 0.18+)
- `--guided-decoding-backend` (legacy, still accepted): values `auto`, `xgrammar`, `guidance`, `outlines`, `lm-format-enforcer`
- `--structured-outputs-config.backend` (current canonical form)
- `--structured-outputs-config.enable_in_reasoning=True`
- Default: `auto` — vLLM picks per request. xgrammar is preferred for sustained throughput; guidance for fastest TTFT on complex grammars.

## Notes / breaking changes
- v0.20 default CUDA bumped to 13.0.2; PyTorch 2.11; transformers >= 5.
- `vllm:prompt_tokens_recomputed` metric removed in 0.20.
- Pooler config rename: `logit_bias`/`logit_scale` → `logit_mean`/`logit_sigma` (0.20).
- CUDAGraph memory profiling enabled by default in 0.20.
