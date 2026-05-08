---
tags: [infrastructure, serving, sglang]
last_updated: 2026-05-08
source_count: 2
---

# SGLang

The strongest open-source alternative to [[infrastructure/vllm]] in 2026,
particularly for prefix-heavy, agentic, RAG, multi-turn, and structured-output
workloads. Apache-2.0. Adopted by xAI, AMD, NVIDIA, Intel, LinkedIn, Cursor,
Oracle Cloud, Google Cloud, Microsoft Azure, AWS — "trillions of tokens in
production each day." [Source: [[sources/sglang-distributed-2026-05]]]

## What it does differently

Two distinguishing innovations make SGLang competitive with — and on specific
workloads measurably better than — vLLM:

### RadixAttention

LRU radix tree of KV cache blocks across requests. Automatically reuses KV
for any shared prefix between any pair of in-flight requests. Where vLLM's
prefix caching is per-engine and request-bounded, SGLang's is global and
persistent across sessions.

> Paper claim (RadixAttention, Zheng et al.): "**up to 6.4× higher
> throughput compared to existing inference systems**" on agent control,
> logical reasoning, RAG, and multi-turn conversations.

This matters most for tool-calling / agent workloads, where every turn shares
a long system prompt + tool schema + prior conversation prefix. With vLLM you
get prefix caching only if the request hits the same engine instance; with
SGLang RadixAttention the cache is engine-wide.

### Compressed finite state machine (CFSM)

Constrained decoding (JSON / regex / grammar) with "**3× faster JSON
decoding**" vs xgrammar-based competitors. Useful when tool-call format
compliance is the bottleneck.

## Tool-calling parsers (`--tool-call-parser`)

[Source: [[sources/sglang-distributed-2026-05]]]

Comparable to vLLM's parser surface, with separate native names:

| Parser | Models |
|---|---|
| `deepseekv3`, `deepseekv31`, `deepseekv32` | DeepSeek V3 family |
| `glm`, `glm45`, `glm47` | GLM-4 / 4.5 / 4.7 |
| `gpt-oss` | OpenAI gpt-oss-20b / 120b |
| `kimi_k2` | Kimi-K2 family |
| `llama3` | Llama 3.x |
| `llama4` | Llama 4 |
| `mistral` | Mistral / Devstral / Codestral |
| `qwen`, `qwen25`, `qwen3_coder` | Qwen2.5 / Qwen3 / Qwen3-Coder |
| `pythonic` | pythonic-format models |
| `step3` | StepFun Step 3 |

Plus a **separate** `--reasoning-parser` (`deepseek-v3`, `qwen3`, `kimi_k2`)
for thinking-mode models — splits `<think>...</think>` content from the
final answer.

Day-0 model support is a stated commitment: "MiMo-V2-Flash, Nemotron 3 Nano,
Mistral Large 3, LLaDA 2.0 Diffusion LLM, MiniMax M2."

## Distributed serving

| Flag | Meaning |
|---|---|
| `--tp N` | Tensor parallel size |
| `--dp N` | Data parallel size |
| `--pp N` | Pipeline parallel size |
| `--enable-dp-attention` | DP attention + EP/TP MoE pattern (DeepSeek-class) |
| `--nnodes`, `--node-rank`, `--dist-init-addr` | Multi-node coordination |

### Multi-node recipes

```bash
# 2× H200×8 — DeepSeek-V3 with DP attention + EP MoE
python -m sglang.launch_server \
  --model deepseek-ai/DeepSeek-V3-0324 \
  --enable-dp-attention --tp 16 --dp 2 \
  --nnodes 2 --node-rank 0 --dist-init-addr <HEAD>:5000

# 8× H200 single node
python -m sglang.launch_server \
  --model ... --enable-dp-attention --tp 8 --dp 8
```

### Pipeline parallelism (LMSYS Jan 2026 breakthrough)

SGLang PP combines **Chunked Pipeline Parallelism (CPP)**, **Async P2P
Communication**, and **Dynamic Chunking**. CPP partitions prompts into 4–6 K
token chunks so stage 1 can advance to chunk 2 while stage 2 processes chunk 1.

Reported result (DeepSeek-V3.1):

> "**PP4 TP8 with this implementation yields a 3.31× Prefill Throughput**"
> vs TP8.

> Outperforms TP32 (2.54×) by 30.5%. Maintains ">80% scaling efficiency."

Qwen3-235B-A22B-FP8: PP8 reduces TTFT from 55.5 s → 10.5 s (~81% improvement).

**Why this matters for AWS**: comm volume on PP is `B·S·H·(P-1)·bytes` per
step, vs TP's `4·B·S·H·L·bytes` per layer — near-order-of-magnitude lower.
On nodes without NVLink (g5/g6/g6e on AWS lack NVLink at instance boundary),
PP across nodes is the right choice. SGLang's PP scales further than
vLLM's PP today on these substrates.

## Quantization

FP4 / FP8 / INT4 / AWQ / GPTQ. (No mainline support for AQLM, BitNet, QuIP#,
ExLlamaV3 — those are [[infrastructure/serving-stack-landscape|Aphrodite]]
specialties.)

## Benchmarking — `bench_serving`

`python -m sglang.bench_serving` mirrors `vllm bench serve`:

```bash
python3 -m sglang.bench_serving \
  --backend sglang --host 127.0.0.1 --port 30000 \
  --num-prompts 1000 \
  --model meta-llama/Llama-3.1-8B-Instruct
```

Backends: `sglang` (native), `sglang-oai`, `vllm`, `lmdeploy`, `trt`, `truss`
— **single harness against multiple engines**, useful for cross-engine
comparisons.

Datasets: `sharegpt`, `random`, `image`, `generated-shared-prefix`, `mmmu`.
The `generated-shared-prefix` dataset specifically exercises prefix-cache wins;
use it to measure RadixAttention impact.

`--flush-cache` for cold/warm cache comparisons. See
[[concepts/serving-performance-measurement]] for the cold-vs-warm trap.

## SGLang vs vLLM — when to pick which

[Source: [[sources/sglang-distributed-2026-05]]]

| Workload | Pick |
|---|---|
| Agent / tool-calling / RAG / multi-turn (long shared prefix) | **SGLang** (RadixAttention) |
| Strict JSON / grammar compliance bottleneck | **SGLang** (CFSM, "3× faster JSON") |
| High concurrency, mostly-cold-cache chat | **vLLM** (C++ router avoids GIL contention) |
| Day-0 newest model on release day | Either — both ship fast |
| AWQ-W4A16 only, max throughput on small GPU | **LMDeploy** beats both |
| Disaggregated prefill/decode at scale | Either, under [[infrastructure/nvidia-dynamo]] |

⚠ **Always run both harnesses against both engines** when publishing a
comparison: a `vllm bench serve` run against SGLang underestimates SGLang's
prefix-cache gains; an `sglang.bench_serving` run against vLLM with
`generated-shared-prefix` overestimates SGLang relative gain. Report both.

## A10G g5.xlarge fit

Same model footprint as vLLM (single replica; no model-loading benefit).
**RadixAttention's gains are workload-dependent**: prefix-heavy tool-calling
agents see large wins; one-shot chat sees marginal differences. If your
workload is BFCL-style multi-turn agentic, SGLang on a single g5.xlarge
should outperform vLLM on the same hardware. Verify with
`sglang.bench_serving --dataset-name generated-shared-prefix`.

## Related

- [[infrastructure/vllm]]
- [[infrastructure/serving-stack-landscape]]
- [[infrastructure/nvidia-dynamo]] (Dynamo can use SGLang as a backend)
- [[concepts/parallelism-strategies]]
- [[concepts/serving-performance-measurement]]
- [[comparisons/serving-stacks-comparison]]

## Sources

- [[sources/sglang-distributed-2026-05]]
- [[sources/serving-stacks-alternatives-2026-05]]
