---
tags: [source, infrastructure, sglang]
source_path: raw/sglang-distributed-2026-05.md
source_url: https://github.com/sgl-project/sglang
ingested: 2026-05-08
last_updated: 2026-05-08
---

# SGLang multi-node + bench_serving (May 2026)

SGLang's distributed-inference flags, the chunked-pipeline-parallelism
breakthrough (LMSYS Jan 2026), tool-parser surface, and the bench_serving
harness used for engine-vs-engine comparisons.

## Key claims

### Project positioning

- "high-performance serving framework for large language models and
  multimodal models" (README) — Apache-2.0.
- "low-latency and high-throughput inference across a wide range of setups,
  from a single GPU to large distributed clusters."
- Adopters listed: "xAI, AMD, NVIDIA, Intel, LinkedIn, Cursor, Oracle Cloud,
  Google Cloud, Microsoft Azure, AWS." "trillions of tokens in production
  each day."

### Distinguishing innovations

- **RadixAttention** (paper https://arxiv.org/abs/2312.07104): LRU radix-tree
  KV cache across requests. Paper claim: "up to 6.4× higher throughput
  compared to existing inference systems" on agent control, logical
  reasoning, RAG, multi-turn.
- **Compressed finite state machine** for constrained decoding — "3× faster
  JSON decoding."

### Multi-node flags

- `--tp` (tensor parallel)
- `--dp` (data parallel)
- `--pp` (pipeline parallel)
- `--nnodes`, `--node-rank`, `--dist-init-addr`
- `--enable-dp-attention` — DP attention + EP/TP MoE pattern (DeepSeek)

### Recipes

- 2× H200×8, DeepSeek-V3 with DP attention + EP MoE:
  `--enable-dp-attention --tp 16 --dp 2`.
- 8× H200, single node: `--enable-dp-attention --tp 8 --dp 8`.

### Pipeline parallelism (LMSYS blog, 2026-01-15)

Three techniques: **Chunked Pipeline Parallelism (CPP)**, **Async P2P
Communication**, **Dynamic Chunking**.

Performance (DeepSeek-V3.1):
- "PP4 TP8 with this implementation yields a **3.31× Prefill Throughput**"
  vs TP8.
- Outperforms TP32 (2.54×) by 30.5%.
- Maintains ">80% scaling efficiency."

Qwen3-235B-A22B-FP8: PP8 reduces TTFT from 55.5 s → 10.5 s (~81% improvement).

### Tool-call parsers (May 2026)

`--tool-call-parser`: deepseekv3, deepseekv31, deepseekv32, glm, glm45,
glm47, gpt-oss, kimi_k2, llama3, llama4, mistral, pythonic, qwen, qwen25,
qwen3_coder, step3.

Separate `--reasoning-parser` (deepseek-v3, qwen3, kimi_k2) for thinking-mode
models.

Day-0 model support: "MiMo-V2-Flash, Nemotron 3 Nano, Mistral Large 3,
LLaDA 2.0 Diffusion LLM, MiniMax M2."

### bench_serving harness

- `python -m sglang.bench_serving` — mirrors `vllm bench serve`.
- Backends: `sglang` (native), `sglang-oai`, `vllm`, `lmdeploy`, `trt`,
  `truss` — single harness against multiple engines.
- Datasets: `sharegpt`, `random`, `image`, `generated-shared-prefix`, `mmmu`.
- `generated-shared-prefix` specifically exercises prefix-cache wins — useful
  for measuring RadixAttention impact.
- `--flush-cache` flag for cold/warm cache comparisons.

### Cross-engine comparison gotcha

Third-party benchmarks (kanerika.com, particula.tech, runpod.io,
spheron.network):
- SGLang ~29% throughput edge over vLLM in steady state on H100.
- SGLang RadixAttention wins on prefix-heavy workloads (~16,200 vs
  ~12,500 tok/s in one test).
- vLLM's C++ router wins under high concurrency due to GIL contention in
  SGLang's Python router.
- "vLLM shows noticeable degradation at batch sizes of 8 and above" under
  guided decoding; SGLang less affected.

⚠ **Always run both harnesses against both engines** for any published
comparison.

### Quantization

FP4/FP8/INT4/AWQ/GPTQ.

### Citation URLs

- https://github.com/sgl-project/sglang
- https://docs.sglang.io/
- https://docs.sglang.io/advanced_features/tool_parser.html
- https://docs.sglang.io/developer_guide/bench_serving.html
- https://www.lmsys.org/blog/2026-01-15-chunked-pipeline/
- https://github.com/sgl-project/sglang/blob/main/docs/basic_usage/deepseek_v3.md
- https://arxiv.org/abs/2312.07104

## Pages updated on ingest

- [[infrastructure/sglang]] (NEW)
- [[infrastructure/serving-stack-landscape]] (NEW)
- [[concepts/parallelism-strategies]] (NEW)
- [[concepts/serving-performance-measurement]] (NEW)
- [[comparisons/serving-stacks-comparison]] (NEW)
