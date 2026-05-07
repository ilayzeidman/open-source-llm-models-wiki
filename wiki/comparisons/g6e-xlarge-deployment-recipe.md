---
tags: [comparison, deployment, recipe, g6e, l40s]
last_updated: 2026-05-07
source_count: 5
---

# g6e.xlarge deployment recipe — Qwen3-Coder-30B-A3B + Devstral-Small

A copy-paste-ready deployment spec for the wiki's recommended next step up from
[[hardware/a10g-g5xlarge]]. Hand this to an agent or a teammate; everything
needed to bring up either model on a single L40S is below.

See [[comparisons/models-by-budget]] Tier 2 for the broader rationale and
[[hardware/g6e-l40s]] for the hardware deep dive.

## Target

| Field | Value |
|---|---|
| AWS instance | `g6e.xlarge` (us-east-1, on-demand $1.861/hr) |
| GPU | 1× NVIDIA L40S, 48 GB GDDR6, sm_89 (Ada Lovelace) |
| Memory bandwidth | 864 GB/s |
| Tensor cores | FP8 native (no MXFP4) |
| Host | 4 vCPU / 32 GiB RAM |
| Serving stack | vLLM ≥ 0.10.0 (no NVIDIA Dynamo — single-GPU; see [[infrastructure/nvidia-dynamo]]) |
| OS image | Deep Learning AMI (Ubuntu 22.04), NVIDIA driver ≥ 550, CUDA 12.4 |

## Operating model

Serve **one model at a time** on the single GPU. Redeploy to swap. Do **not**
co-locate — Qwen3-Coder-30B-A3B FP8 alone uses ~31 GB of the 48 GB.

## Primary model — daily driver (tool calling + general code generation)

[Source: [[sources/qwen3-coder-cards]]]

| Field | Value |
|---|---|
| HF repo | `Qwen/Qwen3-Coder-30B-A3B-Instruct` |
| Architecture | MoE, 31B total / 3.3B active |
| License | Apache-2.0 |
| Quant | FP8 (~31 GB weights; full 256K context fits) |
| vLLM parser | `qwen3_coder` (mainline, vLLM ≥ 0.10.0) |

Launch:

```bash
vllm serve Qwen/Qwen3-Coder-30B-A3B-Instruct \
  --enable-auto-tool-choice --tool-call-parser qwen3_coder \
  --quantization fp8 \
  --max-model-len 32768
```

Sampling defaults: `T=0.7, top_p=0.8, top_k=20, repetition_penalty=1.05`
(loaded automatically from the model's `generation_config.json`).

> ⚠ **`--max-model-len` ceiling on this hardware.** The model's architectural
> max is 256 K, but the *serveable* ceiling on a single L40S 48 GB after FP8
> weights (~29 GB) and vLLM framework overhead (~9 GB) is bounded by the
> ~10 GB KV-cache budget that's left. At Qwen3-Coder-30B's 96 KB/token
> KV size, that's ~110 K tokens upper-bound, but vLLM's `_check_enough_kv_cache_memory`
> reserves slack — **131 K rejects engine startup; 32 K is the realistic
> ceiling for FP8 on this GPU.** See [[concepts/kv-cache-and-context-length]]
> for the math.
>
> If you need >32 K usable context on Qwen3-Coder-30B, jump to a single
> A100 80 GB (or larger), or use AWQ INT4 instead of FP8 (halves weights
> and roughly doubles the KV budget).

See [[models/qwen3-coder-30b-a3b]] for full details.

## Secondary model — agentic SWE-bench / repo-edit workloads

[Source: [[sources/devstral-small-cards]]]

| Field | Value |
|---|---|
| HF repo | `mistralai/Devstral-Small-2-24B-Instruct-2512` (or `-2507` for the SWE-V 53.6 baseline) |
| Architecture | dense 24B |
| License | Apache-2.0 |
| Quant | FP8 (~24 GB weights; comfortable KV headroom) |
| vLLM parser | `mistral` |

> ⚠ **Do not use FP16 on a single L40S.** Per [[models/devstral-small]], FP16
> "just fits" weights with no KV-cache headroom. FP8 is the practical choice.

Launch:

```bash
vllm serve mistralai/Devstral-Small-2-24B-Instruct-2512 \
  --tokenizer_mode mistral --config_format mistral --load_format mistral \
  --enable-auto-tool-choice --tool-call-parser mistral \
  --quantization fp8 \
  --max-model-len 32768
```

See [[models/devstral-small]] for variants (including the 123B flagship that
needs g6e.12xlarge).

## Why this combo

[Source: [[sources/aws-extended-gpu-pricing-2026-05]], [[sources/nvidia-l4-l40s-specs]],
[[sources/vllm-tool-calling-docs]]]

- 48 GB single-GPU eliminates the AWQ INT4 quality compromise required on g5.xlarge (24 GB).
- FP8 native on Ada (sm_89) — no software dequantization tax.
- Both models are Apache-2.0 — zero license risk.
- Both have first-class mainline vLLM tool-call parsers (`qwen3_coder`, `mistral`).
- Single-GPU avoids the PCIe tensor-parallel comms overhead of g5.12xlarge / g6e.12xlarge.

## Verification checklist (after deploy)

1. `nvidia-smi` shows weights + pre-allocated KV cache pool. Expect roughly
   ~42 GB used on Qwen3-Coder-30B FP8 with `--max-model-len 32768` (29 GB
   weights + ~10 GB pre-allocated KV pool + workspace) and ~30 GB on Devstral
   FP8. **2× the model-weight number indicates FP8 dequantization fallback** —
   investigate before accepting; should not happen on Ada/Hopper.
2. POST a tool-calling test to `/v1/chat/completions` with `tools=[...]`; confirm
   the response includes a structured `tool_calls` field, not a raw string in `content`.
3. Long-context smoke test: send a prompt close to (but under) `--max-model-len`;
   confirm no truncation or context-length error and that `usage.prompt_tokens`
   reflects the actual long input. At `--max-model-len 32768`, a ~26 K-token
   prompt is a good test.

### When the readiness probe lies

If you front the worker with `dynamo.frontend`, `GET /v1/models` returns
HTTP 200 with `{"data":[]}` from the moment the frontend is up — long
before the worker has loaded weights. A useful readiness probe must
require the data array to be non-empty (e.g. `grep -q '"id"'`) — see
[[infrastructure/nvidia-dynamo#operational-gotcha-v1models-answers-empty-during-startup]].

## Notes for the operator

- `--quantization fp8` performs **dynamic** FP8 quantization at load time. For
  faster cold-start and lower CPU peak, look for a pre-quantized
  `Qwen3-Coder-30B-A3B-Instruct-FP8` mirror on HF and drop the flag.
- `g6e.xlarge` only has 4 vCPU / 32 GiB RAM. If you batch heavily or do
  client-side preprocessing, jump to `g6e.2xlarge` ($2.242/hr) — same single
  L40S, more host CPU/RAM. See [[hardware/g6e-l40s]].

## Related

- [[wiki/overview]]
- [[hardware/g6e-l40s]]
- [[hardware/a10g-g5xlarge]] (the box this recipe upgrades from)
- [[models/qwen3-coder-30b-a3b]]
- [[models/devstral-small]]
- [[infrastructure/vllm]]
- [[infrastructure/nvidia-dynamo]]
- [[infrastructure/quantization]]
- [[concepts/kv-cache-and-context-length]]
- [[comparisons/models-by-budget]]

## Sources

- [[sources/aws-extended-gpu-pricing-2026-05]]
- [[sources/nvidia-l4-l40s-specs]]
- [[sources/qwen3-coder-cards]]
- [[sources/devstral-small-cards]]
- [[sources/vllm-tool-calling-docs]]
