# SGLang multi-node — May 2026

## Authoritative URLs

- Project: https://github.com/sgl-project/sglang
- Docs: https://docs.sglang.io/
- Pipeline parallelism blog (LMSYS, 2026-01-15):
  https://www.lmsys.org/blog/2026-01-15-chunked-pipeline/
- DeepSeek-V3 multi-node guide:
  https://github.com/sgl-project/sglang/blob/main/docs/basic_usage/deepseek_v3.md
- Tool parser: https://docs.sglang.io/advanced_features/tool_parser.html
- Bench serving: https://docs.sglang.io/developer_guide/bench_serving.html

## Key flags

- `--tp` (tensor parallel)
- `--dp` (data parallel)
- `--pp` (pipeline parallel)
- `--nnodes`, `--node-rank`, `--dist-init-addr`
- `--enable-dp-attention` — DP attention + EP/TP MoE pattern (DeepSeek)

## Multi-node recipes

- 2× H200×8, DeepSeek-V3 with DP attention + EP MoE:
  `--enable-dp-attention --tp 16 --dp 2`
- 8× H200, single node:
  `--enable-dp-attention --tp 8 --dp 8`

## Pipeline parallelism (LMSYS blog headline: "Scaling to Million-Token Contexts")

SGLang's PP combines three techniques: **Chunked Pipeline Parallelism (CPP)**,
**Asynchronous P2P Communication**, and **Dynamic Chunking**.

- CPP: "partitions the prompt into smaller chunks (e.g., 4K or 6K tokens)" so
  stage 1 can move to chunk 2 while stage 2 processes chunk 1.
- Async P2P returns "a P2PWork handle" without GPU-side blocking.
- Dynamic chunking models "the cumulative runtime as a quadratic function of
  sequence length" to keep per-chunk times equal.

### Performance
- "PP4 TP8 with this implementation yields a **3.31× Prefill Throughput** for
  DeepSeek-V3.1" vs TP8.
- Outperforms TP32 (2.54×) by 30.5%.
- Qwen3-235B-A22B-FP8: PP8 reduces TTFT from 55.5s → 10.5s (~81% improvement).
- Maintains ">80% scaling efficiency."

### Comm volume comparison
- TP: `4·B·S·H·L·bytes`
- PP: `B·S·H·(P−1)·bytes`
- Near order-of-magnitude reduction at multi-node scale.
- **Strongest argument for PP across nodes when the link is slow** (no NVLink/
  no EFA, e.g. g5).

## Production scale
"trillions of tokens in production each day" across xAI, AMD, NVIDIA, Intel,
LinkedIn, Cursor, Oracle Cloud, Google Cloud, Microsoft Azure, AWS.

## bench_serving harness

`python -m sglang.bench_serving` — intentionally mirrors `vllm bench serve`.

### Backends
`sglang` (native `/generate`), `sglang-oai` / `vllm` / `lmdeploy` (OpenAI-
compatible completions), chat variants, `trt` (TensorRT-LLM), `truss`.

### Datasets
`sharegpt`, `random`, `image`, `generated-shared-prefix`, `mmmu`.

The `generated-shared-prefix` dataset is specifically designed to exercise
**prefix-caching wins** (synthetic prompts that share a long prefix) — useful
for measuring RadixAttention impact.

### Key flags
```
--backend, --host, --port  / --base-url
--model                    auto-detected if /v1/models is exposed
--num-prompts
--dataset-name             sharegpt|random|image|generated-shared-prefix|mmmu
--random-input-len, --random-output-len
--sharegpt-output-len
--request-rate             "inf" for burst
--max-concurrency
--disable-stream
--output-file              JSONL of per-request data
--output-details           include per-token arrays
--profile, --flush-cache   server profiling and cache control
```

### Quick example
```bash
python3 -m sglang.bench_serving \
  --backend sglang --host 127.0.0.1 --port 30000 \
  --num-prompts 1000 \
  --model meta-llama/Llama-3.1-8B-Instruct
```

Outputs: req/s, input/output token throughput, TTFT/ITL p50/p95/p99, e2e
latency. The `--flush-cache` flag is critical for cold/warm cache comparisons.

### Cross-engine comparison gotcha
Third-party benchmarks find SGLang's RadixAttention typically beats vLLM on
prefix-heavy workloads (~16,200 vs ~12,500 tok/s in one test) while vLLM's C++
router wins on cold-cache high-concurrency loads. **Always run both harnesses
against both engines** if you're publishing a comparison.
