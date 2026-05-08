---
tags: [source, concepts, benchmarks, tools, performance]
source_path: raw/llm-serving-perf-tools-2026-05.md
ingested: 2026-05-08
last_updated: 2026-05-08
---

# LLM serving benchmarking tools (May 2026)

The toolset for measuring serving performance — `vllm bench serve`,
NVIDIA GenAI-Perf/AIPerf, LLMPerf, SGLang `bench_serving`, MLPerf v5.0,
plus workload datasets and the common traps.

## Key claims

### Tool comparison

| Tool | Mode | Streaming | Open / Closed loop | Tokenizer convention |
|---|---|---|---|---|
| `vllm bench serve` | CLI in vLLM | yes (SSE) | both | Model's own |
| **NVIDIA GenAI-Perf** (and successor AIPerf) | Standalone client | yes | both (`--concurrency` vs `--request-rate`) | Configurable |
| **LLMPerf** (archived Dec 2025) | Ray-based | yes | closed | Llama-2 fast tokenizer |
| **SGLang `bench_serving`** | Module | yes | both | Model's own |
| **NVIDIA Triton perf_analyzer** | Older | no SSE | both | n/a (per-backend) |
| **MLPerf Inference v5.0 Server** | Standardized | yes | open (Poisson) | Per benchmark spec |
| **k6 / Locust** | Generic load tools | poor | varies | Bring your own |

### `vllm bench serve` (formerly `benchmark_serving.py`)

- Subcommands: `latency`, `serve`, `throughput`.
- Datasets: `sharegpt`, `sonnet`, `random`, `random-mm`, `hf`, `custom`,
  `prefix_repetition`, `spec_bench`, `speed_bench`.
- Key flags: `--num-prompts`, `--request-rate`, `--max-concurrency`,
  `--num-warmups`, `--save-result`.
- Outputs: per-request TTFT, ITL, e2e latency, tokens, p50/p95/p99
  distributions.

### NVIDIA GenAI-Perf

- "client-side" generative-AI benchmarking; works against any OpenAI-
  compatible endpoint (vLLM, SGLang, TGI, NIM, Triton).
- Recommended sweep: `for concurrency in 1 2 5 10 50 100 250`.
- `--measurement-interval` must be long enough for several requests to finish
  at every concurrency. NVIDIA: "for Llama-3 70B at 250 concurrency, 100,000 ms."
- "We recommend starting a GenAI-Perf container on the same server as NIM
  to avoid network latency."
- Successor: **AIPerf** (https://github.com/ai-dynamo/aiperf) — CLI largely
  compatible.

### LLMPerf (archived)

- GitHub repo archived Dec 17 2025. Methodology still cited.
- Default workload: ISL 550 (stdev 150) / OSL 150 (stdev 20) — based on real
  Anyscale Endpoints traffic.
- Closed-loop with round-based concurrency (under-utilizes between rounds).
- **TTFT included in ITL** (different from GenAI-Perf).
- Anyscale tokenizer policy: always Llama-2 fast tokenizer for cross-system
  fairness.

### SGLang `bench_serving`

- Mirrors `vllm bench serve`.
- Backends: `sglang`, `sglang-oai`, `vllm`, `lmdeploy`, `trt`, `truss` —
  single harness, multiple engines.
- `generated-shared-prefix` dataset specifically exercises prefix-cache wins.
- `--flush-cache` for cold/warm comparisons.

### MLPerf Inference v5.0 — LLM track

- Llama 3.1 405B (LongBench/Ruler/GovReport mix) — 99p TTFT ≤ 6s,
  99p TPOT ≤ 175 ms.
- Llama 2 70B Interactive — 99p TPOT ≤ 40 ms, 99p TTFT ≤ 450 ms.
- Server scenario: open-loop Poisson arrivals.
- Reference impl uses vLLM.
- Gold standard for cross-organization comparison.

### Workload datasets

- **ShareGPT** — `ShareGPT_V3_unfiltered_cleaned_split.json`. Standard chat
  workload. Caveat: "current implementation only uses the first 2 lines of
  each dialog" — multi-turn fidelity lost.
- **LongBench** — long-context; adopted by MLPerf v5.0.
- **BurstGPT** — Microsoft Azure OpenAI traces. 5.29M+ traces over 121 days.
  arXiv: https://arxiv.org/abs/2401.17644. Most realistic public LLM trace.
- **Alpaca** — short instruction (prefill-heavy).
- **vLLM/SGLang `random`** — fixed-length synthetic; for engine comparison
  only, not capacity sizing.
- **No widely-adopted public agentic-tool-call trace.** Closest: BFCL v3/v4,
  AgentBoard, MCP-Bench, MINT — all synthetic. Production-grade trace gap.

### Benchmark traps (consolidated)

1. **Cold vs warm cache** — first 10–100 requests have abnormally high TTFT.
   Always warm up. SqueezeBits study: "throughput reduction of ~36.7% and a
   TPOT increase of ~25.0%" if prefix caching on but no shared prefix.
2. **Batch-1 vs realistic concurrency** — sweep 1, 2, 5, 10, 50, 100, 250.
3. **Open-loop vs closed-loop** — use open-loop for SLO sizing.
4. **Tokenizer skew** — Llama 2 SentencePiece ~12% more tokens than ChatGPT
   for same English. Use one standardized tokenizer for engine comparison;
   model's own for $/Mtok.
5. **GPU-Util ≠ throughput** — `nvidia-smi` shows 100% during memory-bound
   decode that leaves SMs idle. Use MFU (compute-bound) and MBU
   (memory-bound) instead.
6. **Streaming on/off matters** — non-streaming has no meaningful TTFT.
7. **`ignore_eos: true`** for fixed-length output benchmarks.
8. **Network locality** — co-locate client with server unless measuring
   network.
9. **`--measurement-interval`** must be long enough for ≥ several requests
   to complete at each concurrency.
10. **Variance** — run ≥ 3 times; report mean ± stdev. Cloud GPU variance is
    real (Databricks: 2× across providers for "the same" 8×A100).

### Citation URLs

- https://github.com/vllm-project/vllm/blob/main/benchmarks/README.md
- https://docs.vllm.ai/en/latest/benchmarking/cli/
- https://docs.vllm.ai/en/stable/cli/bench/serve/
- https://developer.nvidia.com/blog/llm-performance-benchmarking-measuring-nvidia-nim-performance-with-genai-perf/
- https://developer.nvidia.com/blog/measuring-generative-ai-model-performance-using-nvidia-genai-perf-and-an-openai-compatible-api/
- https://docs.nvidia.com/nim/benchmarking/llm/latest/metrics.html
- https://github.com/ai-dynamo/aiperf
- https://github.com/ray-project/llmperf
- https://docs.sglang.io/developer_guide/bench_serving.html
- https://mlcommons.org/2025/04/llm-inference-v5/
- https://github.com/mlcommons/inference
- https://developer.nvidia.com/blog/nvidia-blackwell-delivers-massive-performance-leaps-in-mlperf-inference-v5-0/
- https://github.com/HPMLL/BurstGPT
- https://arxiv.org/abs/2401.17644
- https://huggingface.co/blog/tgi-benchmarking
- https://huggingface.co/blog/martinigoyanes/llm-inference-at-scale-with-tgi
- https://www.truefoundry.com/blog/llm-locust-a-tool-for-benchmarking-llm-performance

## Pages updated on ingest

- [[concepts/serving-performance-measurement]] (NEW)
- [[concepts/benchmarks]]
- [[comparisons/scaling-1-to-5-machines]] (NEW)
