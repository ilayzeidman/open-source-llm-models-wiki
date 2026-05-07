---
tags: [concepts, serving, memory, vllm]
last_updated: 2026-05-07
source_count: 1
---

# KV cache and the real context-length ceiling

The headline `max_position_embeddings` (or "256K context") in a model card is
*architectural* — it's what the attention mechanism can address. The
*serveable* context length on a given GPU is almost always smaller, and is
gated by **KV-cache memory**, not by total VRAM.

This page explains why, gives the formula, and shows how to pick a safe
`--max-model-len` from first principles.

## Three buckets, fixed priority

When [[infrastructure/vllm]] (or any KV-paged engine) loads a model, VRAM is
carved into three regions, allocated in this order:

1. **Weights** — the model itself, fixed once loaded.
2. **Framework overhead** — CUDA graph workspace, torch.compile cache,
   activations, headroom. Empirically ~6–10 GB on a single GPU regardless of
   model size; bigger for multi-GPU.
3. **KV cache** — whatever's left. Per-token, per-layer, per-request.

KV cache is the only bucket that *grows* with usage. Doubling
`--max-model-len` doubles the per-request KV footprint.

```
┌───────────────────────────────────────────────────────────┐
│                       GPU VRAM (total)                    │
├───────────────────────────────────────────────────────────┤
│ [1] Weights (fixed)                                       │
│ [2] Framework overhead (~6–10 GB)                         │
│ [3] KV cache  ← whatever's left; this is the lever        │
└───────────────────────────────────────────────────────────┘
```

## Per-token KV size

For each token in a request, at each layer, the attention layer writes
**K**ey and **V**alue tensors:

```
KV_per_token  =  num_layers
              ×  2                       # (K + V)
              ×  num_kv_heads            # GQA reduces this vs num_attention_heads
              ×  head_dim
              ×  bytes_per_element       # KV dtype: fp16 → 2; fp8 → 1
```

The fields come from the model's `config.json` — `num_hidden_layers`,
`num_key_value_heads`, `head_dim` (or `hidden_size / num_attention_heads`).
Models that use Grouped-Query Attention (GQA) with few KV heads have
*much* smaller KV caches than their attention-head count would suggest.

### Worked examples

[Source: model `config.json` files for each]

| Model | Layers | KV heads | head_dim | KV dtype | KV per token |
|---|---:|---:|---:|---|---:|
| Llama-3.1-8B-Instruct | 32 | 8 | 128 | fp16 | **128 KB** |
| Llama-3.3-70B-Instruct | 80 | 8 | 128 | fp16 | **320 KB** |
| Qwen2.5-Coder-7B-Instruct | 28 | 4 | 128 | fp16 | **56 KB** |
| Qwen2.5-Coder-32B-Instruct | 64 | 8 | 128 | fp16 | **256 KB** |
| Qwen3-Coder-30B-A3B-Instruct | 48 | 4 | 128 | fp16 | **96 KB** |
| Mistral-Small-24B | 40 | 8 | 128 | fp16 | **160 KB** |
| Devstral-Small-24B | 40 | 8 | 128 | fp16 | **160 KB** |

(Numbers above use `KV per token = layers × 2 × kv_heads × head_dim × 2`.)

The MoE Qwen3-Coder-30B-A3B has a **smaller per-token KV than the dense
Qwen2.5-Coder-32B** (96 KB vs 256 KB) despite both being ~30 B
parameters — because it has fewer layers and fewer KV heads. MoE generally
helps KV memory.

## The startup gate

vLLM's V1 engine performs an explicit check before serving:

> `_check_enough_kv_cache_memory`: does the available KV-cache budget
> hold at least one full-length request (`max_model_len`)?

If the answer is no, the engine **refuses to start** — not at request time,
at startup. Container orchestration (`--restart unless-stopped`) will then
loop the same failure indefinitely if the value is too high. Surface symptom:
`/v1/models` answers with `{"data": []}` forever; only the worker container
shows the error. See [[infrastructure/nvidia-dynamo#operational-gotcha-v1models-answers-empty-during-startup]].

## Picking a safe `--max-model-len`

Napkin formula:

```
KV_budget   ≈  vram_total - weight_memory - 9 GB framework_overhead
max_safe    =  KV_budget  /  KV_per_token
```

Then **use ~70 % of `max_safe`** to leave headroom for vLLM's graph
workspace, prefix-cache slack, and concurrent requests. The full computed
value is the absolute upper bound, not a target.

### Worked example: Qwen3-Coder-30B-A3B FP8 on L40S 48 GB

[Source: vLLM 0.16+ engine logs from a g6e.xlarge deploy.]

```
weight_memory  ≈  29 GB    (FP8 dynamic quant of 31 B params)
overhead       ≈   9 GB    (CUDA graphs + torch.compile)
KV_budget      ≈  10 GB
KV_per_token   =  96 KB
max_safe       ≈  10,485 MB / 0.094 MB  ≈  111,500 tokens (upper bound)
recommended    =  ~70 % of 111,500       ≈  78,000 tokens
practical pick =  32,768                  (well within budget; vLLM's own
                                           reservation slack pushes the
                                           true ceiling below the napkin
                                           number)
```

vLLM at startup reports the actual `Available KV cache memory` value
(here ~10 GB) and uses it for the check. The gap between the napkin
number and what vLLM accepts is real — vLLM reserves margin for graph
re-capture and prefix-cache growth.

### When the recipe lists a fallback context

Recipe entries like [[comparisons/g6e-xlarge-deployment-recipe]] cite both
a "headline" max-model-len (the model's architectural max) and an
**OOM fallback** (e.g. 32 K). Treat the fallback as the realistic value
on that exact GPU + quant combo. The headline is only achievable when
the GPU is bigger than the recipe target *or* the model is smaller.

## The four levers when you don't fit

1. **Lower `--max-model-len`** — cheapest. Trade context for survival.
2. **Bigger GPU** — L40S 48 GB → A100 80 GB → H100 80 GB → H200 141 GB.
   See [[hardware/aws-gpu-landscape]].
3. **Smaller model or stronger quantization** — INT4 weight-only halves
   weight memory vs FP8 and frees KV-cache budget. See [[infrastructure/quantization]].
4. **`--kv-cache-dtype fp8`** — halves KV-cache size at a small accuracy
   cost. Supported on Ada/Hopper. Less dramatic than the first three.

Multi-GPU (`--tensor-parallel-size N`) helps weight memory (split across
GPUs) and total KV budget (also split), but adds PCIe / NVLink comms cost.
See [[hardware/multi-gpu-options]].

## Decision tree

```
  Worker fails shortly after "Loading weights took N seconds"?
         │
         ├── log says "_check_enough_kv_cache_memory"?
         │        │
         │        └── yes → KV cache too small at current --max-model-len.
         │                  Apply lever 1, 2, 3, or 4 above.
         │
         └── log says "CUDA out of memory" during weight load?
                  └── weights themselves don't fit. Lever 1 won't help —
                       you need lever 2, 3, or 4.
```

## What `--gpu-memory-utilization` does (and doesn't)

Default 0.92 (vLLM 0.18+) means vLLM may use 92 % of total VRAM. Bumping
to 0.95–0.97 buys you another ~1.5–3 GB on a 48 GB card, which translates
into a few % more `max_safe`. It's a small lever, not a fix for the
"30B FP8 + 131 K context on L40S" scenario, where the gap is ~2 GB and
the recipe asked for ~12 GB.

Don't push above 0.97 — vLLM needs scratch space for graph rebuilds and
cuBLAS workspaces; running too tight causes intermittent OOMs at
serve-time, not startup, which are much harder to diagnose.

## Long-context observations

For context lengths well into the tens of thousands, prefix caching
([[infrastructure/vllm#prefix-caching]]) and chunked prefill have a
much larger impact on perceived TTFT than the absolute `max_model_len`.
A request that uses 28 K tokens with a shared 26 K prefix takes ~prefill
of 2 K plus cache lookup, not 28 K of fresh prefill.

## Related

- [[infrastructure/vllm]]
- [[infrastructure/nvidia-dynamo]]
- [[infrastructure/quantization]]
- [[hardware/aws-gpu-landscape]]
- [[hardware/multi-gpu-options]]
- [[comparisons/g6e-xlarge-deployment-recipe]]

## Sources

- [[sources/vllm-quantization-docs]]
