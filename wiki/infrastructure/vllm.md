---
tags: [infrastructure, serving, vllm]
last_updated: 2026-05-08
source_count: 3
---

# vLLM

High-throughput LLM inference server, OpenAI-compatible API, paged-attention
KV cache. The serving foundation under [[infrastructure/nvidia-dynamo]] and
the wiki's default engine. Alternatives that may fit specific workloads
better: [[infrastructure/sglang]] (RadixAttention; agentic / prefix-heavy),
[[infrastructure/lmdeploy]] (AWQ-W4A16 specialty),
[[infrastructure/tensorrt-llm]] (NVFP4/MXFP4 native on Blackwell). The full
landscape is at [[infrastructure/serving-stack-landscape]].

**Latest stable: vLLM 0.20.1** (released 2026-05-03). Pinning recommendation: `vllm==0.20.1`. [Source: [[sources/vllm-quantization-docs]]]

## What matters for this research

### Tool-call parsing

vLLM ships per-model parsers that translate the model's native tool-call output format into OpenAI-style `tool_calls` JSON. Enable with:

```
--enable-auto-tool-choice --tool-call-parser <name>
```

Custom out-of-tree parsers can be loaded with `--tool-parser-plugin <path>`.

Current parser families (vLLM 0.20.x): [Source: [[sources/vllm-tool-calling-docs]]]

| Parser | Models |
|---|---|
| `hermes` | Hermes-2 Pro / 2 Theta / 3 / 4; Qwen2.5/Qwen3 base instruct |
| `mistral` | Mistral-Instruct-v0.3, Mistral-Small 3.x, Codestral, Devstral |
| `llama3_json` | Llama 3.1 / 3.2 / 3.3 |
| `pythonic` | Llama-3.2 pythonic, ToolACE, Ultravox |
| `llama4_pythonic` | Llama 4 |
| `granite` | Granite 3.0 / 3.1 / 3.3 |
| `granite4` | Granite 4.x |
| `granite-20b-fc` | Granite-20B-FunctionCalling |
| `internlm` | InternLM2.5 |
| `jamba` | AI21 Jamba-1.5 |
| `xlam` | Salesforce/Qwen xLAM v2 family |
| `deepseek_v3` | DeepSeek-V3, DeepSeek-R1 |
| `deepseek_v31` | DeepSeek-V3.1 |
| `qwen3_xml` | Qwen3-Coder |
| `openai` | GPT-OSS |
| `kimi_k2`, `hunyuan_a13b`, `cohere_command3`, `longcat`, `glm45`, `glm47`, `functiongemma`, `olmo3`, `gigachat3`, `minimax` | model-family specific |

Parser names have been purely additive across 2025–2026 — no renames or deletions.

### Per-model parser table (the 13 wiki candidates)

| Model | Parser | Status | Source |
|---|---|---|---|
| Qwen2.5 / Qwen2.5-Coder | `hermes` | ⚠ **Known-flaky on Coder.** Coder emits ```json``` fences, not `<tool_call>` tags. vLLM issues #10952, #29192, #32926; community parser PR #32931 unmerged. | [[sources/vllm-tool-calling-docs]] |
| Llama 3.1 / 3.2 / 3.3 | `llama3_json` | Official; chat template required | [[sources/vllm-tool-calling-docs]] |
| Mistral-Small 3.x / Codestral | `mistral` | Official; 3.2 ships improved parallel-call template | [[sources/vllm-tool-calling-docs]] |
| DeepSeek-Coder-V2-Lite | `deepseek_v3` | ⚠ **Unverified for V2** — parser is V3-tuned | [[sources/vllm-tool-calling-docs]] |
| Granite 3.x | `granite` (3.x); `granite4` (4.x) | Official | [[sources/vllm-tool-calling-docs]] |
| Hermes-3 | `hermes` | Canonical target for the parser | [[sources/vllm-tool-calling-docs]] |
| Salesforce xLAM v2 | `xlam` + `xlam_{llama,qwen}` template | Official since vLLM 0.6.5 | [[sources/vllm-tool-calling-docs]] |
| MeetKai Functionary | **No mainline parser.** | Use MeetKai's `server_vllm.py` fork or `--tool-parser-plugin`. | [[sources/vllm-tool-calling-docs]] |
| Phi-3 / Phi-4 | **No official parser.** | Community workaround = `llama3_json` + custom template; brittle. Issues #14682, #15788. Phi-4-mini (3.8B) added native function calling but base Phi-4 14B did not. | [[sources/vllm-tool-calling-docs]] |

### Structured outputs / constrained decoding

Useful when tool-call format compliance is the bottleneck:
- **xgrammar** — vLLM's default since ~late 2024; fast, GBNF-like grammar. Recommended for tool-call schemas.
- **guidance** — preferred when TTFT matters more than steady-state TPS (dense / complex grammars).
- **outlines** — still supported but de-prioritized.
- **lm-format-enforcer** — Python `re` regex dialect; not recommended for new deployments.

CLI: `--guided-decoding-backend {auto,xgrammar,guidance,outlines,lm-format-enforcer}` (legacy). Canonical (0.18+): `--structured-outputs-config.backend=<name>`.

`auto` resolves to xgrammar for almost all tool-call schemas. [Source: [[sources/vllm-quantization-docs]]]

### Quantization backends

vLLM supports loading pre-quantized weights from Hugging Face. See [[infrastructure/quantization]].

| Backend | Format | A10G (Ampere sm_86) | Source |
|---|---|---|---|
| AWQ | INT4 weight-only | ✅ excellent (Marlin kernels) | [[sources/vllm-quantization-docs]] |
| GPTQ | INT4 weight-only | ✅ good (Marlin kernels) | [[sources/vllm-quantization-docs]] |
| INT8 SmoothQuant | W8A8 | ✅ ok (cutlass) | [[sources/vllm-quantization-docs]] |
| FP8 W8A8 | W8A8 | ❌ — Ampere lacks FP8 tensor cores; "compute capability ≥ 8.9 required" | [[sources/vllm-quantization-docs]] |
| FP8 W8A16 | weight-only | ✅ silent fallback (FP8 Marlin) | [[sources/vllm-quantization-docs]] |
| BitsAndBytes | NF4/INT8 | ⚠ slower than AWQ/GPTQ for serving | [[sources/vllm-quantization-docs]] |
| GGUF | various | ✅ supported | [[sources/vllm-quantization-docs]] |

### Prefix caching

`--enable-prefix-caching` is **on by default in V1** (since ~vLLM 0.16). Reuses KV cache across requests sharing a prompt prefix. Big win for tool-calling workloads where the system prompt + tool schemas are repeated every turn.

### Chunked prefill

`--enable-chunked-prefill` is **on by default in V1**. Interleaves prefill and decode in a single batch, smoothing latency.

### Speculative decoding

Smaller draft model proposes tokens, target model verifies. Useful for code generation where hit-rate is high. Vocab compatibility matters — Qwen draft + Qwen target works; cross-family draft pairs often don't.

## Memory tuning flags (current defaults)

[Source: [[sources/vllm-quantization-docs]]]

| Flag | Default | Purpose |
|---|---|---|
| `--gpu-memory-utilization` | **0.92** (was 0.9 prior to ~0.18) | fraction of VRAM vLLM may consume |
| `--max-model-len` | auto | hard ceiling on context length; lowering frees KV-cache budget |
| `--max-num-seqs` | scheduler-dependent | concurrent request cap |
| `--swap-space` | 4 GiB | CPU swap for KV-cache offload |
| `--enforce-eager` | False | disable CUDA graphs (debugging only; ~10–30% slower) |
| `--enable-prefix-caching` | on by default in V1 | reuse KV across shared prefixes |
| `--enable-chunked-prefill` | on by default in V1 | split prefill across iters |
| `--tensor-parallel-size` | 1 | TP group size |
| `--quantization` | None (autodetect) | force a method; explicit `awq_marlin`/`gptq_marlin` on Ampere |
| `--kv-cache-dtype` | auto | `auto`/`fp8`/`fp8_e5m2`/`fp8_e4m3`/`int8` |

### `--max-model-len` is bounded by KV cache, not VRAM

vLLM V1 enforces `_check_enough_kv_cache_memory` at engine startup. If the
configured `--max-model-len` doesn't fit one full-length request in the
KV-cache budget *after* weights and framework overhead, the engine refuses
to start. This is the single most common reason a "fits in VRAM" model
won't serve at its model-card max context. The full math, per-model
KV-per-token table, and the four levers when you don't fit are in
[[concepts/kv-cache-and-context-length]].

## Tensor parallelism

`--tensor-parallel-size N` splits weights across N GPUs. On g5 (PCIe Gen4 only) overhead is meaningful; on p4d/p5 (NVLink) it's nearly free. See [[hardware/multi-gpu-options]].

## Multi-node distributed serving

[Source: [[sources/vllm-distributed-serving-2026-05]]]

vLLM's canonical multi-node rule:

> "**The tensor parallel size should be the number of GPUs in each node, and
> the pipeline parallel size should be the number of nodes.**"

| Flag | Purpose |
|---|---|
| `--tensor-parallel-size N` | Shard each layer across N GPUs (intra-node, NVLink/PCIe) |
| `--pipeline-parallel-size N` | Split layers across N stages (across slow links / multi-node) |
| `--data-parallel-size N` | N independent model replicas |
| `--data-parallel-size-local N` | DP ranks on this node |
| `--data-parallel-address`, `--data-parallel-rpc-port` | Coordination endpoint |
| `--data-parallel-start-rank N` | Rank offset on worker nodes |
| `--data-parallel-backend ray` | Use Ray for DP launch |
| `--enable-expert-parallel` | EP for MoE experts |
| `--all2all-backend ...` | All-to-all kernel (deepep_high_throughput, deepep_low_latency, flashinfer_nvlink_*) |
| `--distributed-executor-backend ray\|mp` | Ray vs multiprocessing |
| `--nnodes`, `--node-rank`, `--master-addr`, `--headless` | Multiprocessing multi-node mode |
| `--api-server-count N` | Scale OAI API workers (large DP) |

**Multi-node recipe (multiprocessing, no Ray):**

```bash
# Head (rank 0)
vllm serve /path/to/model --tensor-parallel-size 8 --pipeline-parallel-size 2 \
  --nnodes 2 --node-rank 0 --master-addr <HEAD_IP>
# Worker (rank 1)
vllm serve /path/to/model --tensor-parallel-size 8 --pipeline-parallel-size 2 \
  --nnodes 2 --node-rank 1 --master-addr <HEAD_IP> --headless
```

For TP/PP/DP/EP decision logic, see [[concepts/parallelism-strategies]].

For when to use **multi-replica DP** (each replica fits a node) vs **multi-node
TP+PP** (model > one node), see [[comparisons/scaling-1-to-5-machines]] —
which also covers the AWS-specific constraint that g5 lacks EFA, making
multi-node TP infeasible on the wiki's primary baseline.

## V1 engine (Jan 2025+, default in vLLM ≥ 0.8.x)

[Source: [[sources/vllm-distributed-serving-2026-05]]]

Architectural changes:
- `EngineCore` execution loop in separate process (multiprocessing).
- Unified scheduler removes prefill/decode distinction — schedule is just
  `{request_id: num_tokens}`.
- Hash-based prefix caching on by default; "less than 1% decrease in
  throughput even when the cache hit rate is 0%."
- Persistent batch with NumPy input prep.
- Symmetric TP architecture (scheduler decoupled from worker 0).
- Piecewise CUDA graphs + FlashAttention 3.

Headline: "V1 achieves up to 1.7× higher throughput compared to V0";
"up to 24% improvement" on generation-heavy workloads (Red Hat blog on 0.8.1).

## 0.20 breaking changes (pin awareness)

[Source: [[sources/vllm-quantization-docs]]]

- PyTorch 2.11, CUDA 13.0.2 default
- **transformers >= 5 required**
- Metric `vllm:prompt_tokens_recomputed` removed → `PrefillStats`
- Pooler config rename: `logit_bias`/`logit_scale` → `logit_mean`/`logit_sigma`
- Petit NVFP4 and Sparse24 quantization paths removed
- `experts_int8` consolidated into FP8 online quantization frontend
- `LLM.reward` deprecated → `LLM.encode`

## Related
- [[infrastructure/nvidia-dynamo]]
- [[infrastructure/serving-stack-landscape]]
- [[infrastructure/sglang]]
- [[infrastructure/vllm-production-stack]]
- [[infrastructure/llm-d]]
- [[infrastructure/quantization]]
- [[concepts/tool-selection]]
- [[concepts/parallelism-strategies]]
- [[concepts/disaggregated-serving]]
- [[concepts/serving-performance-measurement]]
- [[concepts/kv-cache-and-context-length]]
- [[comparisons/scaling-1-to-5-machines]]

## Sources
- [[sources/vllm-tool-calling-docs]]
- [[sources/vllm-quantization-docs]]
- [[sources/vllm-distributed-serving-2026-05]]
