# LLM serving benchmarking tools — May 2026

Consolidated from official READMEs and docs.

## Authoritative URLs

- vLLM benchmarks README: https://github.com/vllm-project/vllm/blob/main/benchmarks/README.md
- vLLM bench CLI docs: https://docs.vllm.ai/en/latest/benchmarking/cli/
- vLLM `bench serve` CLI: https://docs.vllm.ai/en/stable/cli/bench/serve/
- NVIDIA GenAI-Perf (NIM): https://developer.nvidia.com/blog/llm-performance-benchmarking-measuring-nvidia-nim-performance-with-genai-perf/
- NVIDIA GenAI-Perf vs OpenAI: https://developer.nvidia.com/blog/measuring-generative-ai-model-performance-using-nvidia-genai-perf-and-an-openai-compatible-api/
- NVIDIA NIM benchmarking metrics: https://docs.nvidia.com/nim/benchmarking/llm/latest/metrics.html
- AIPerf (GenAI-Perf successor): https://github.com/ai-dynamo/aiperf
- LLMPerf: https://github.com/ray-project/llmperf
- SGLang bench_serving: https://docs.sglang.io/developer_guide/bench_serving.html
- MLPerf v5.0 LLM: https://mlcommons.org/2025/04/llm-inference-v5/
- MLPerf v5.0 results: https://mlcommons.org/2025/04/mlperf-inference-v5-0-results/
- mlcommons/inference: https://github.com/mlcommons/inference
- NVIDIA Blackwell + MLPerf v5.0: https://developer.nvidia.com/blog/nvidia-blackwell-delivers-massive-performance-leaps-in-mlperf-inference-v5-0/

---

## vLLM `vllm bench serve` (formerly `benchmark_serving.py`)

The benchmark scripts moved from `benchmarks/benchmark_serving.py` to a
first-class CLI in vLLM v0.7+. Official entry point: `vllm bench serve`.

### Subcommands
- `vllm bench latency` — single-request latency
- `vllm bench serve` — online server load test (most-used)
- `vllm bench throughput` — offline batch throughput

### Key flags for `vllm bench serve`
```
--model               Model name (auto-fetched from /v1/models if omitted)
--backend             vllm | openai | openai-chat | openai-audio |
                      openai-embeddings | vllm-pooling | vllm-rerank
--base-url            Server URL  (or --host, --port)
--endpoint            Default /v1/completions
--dataset-name        sharegpt | sonnet | random | random-mm | hf |
                      custom | prefix_repetition | spec_bench | speed_bench
--num-prompts         Default 1000
--request-rate        RPS (default inf = saturate)
--max-concurrency     Hard cap on in-flight requests
--num-warmups         Default 0
--save-result --result-dir --result-filename

# Random dataset
--random-input-len    Default 1024
--random-output-len   Default 128
--random-range-ratio  Default 0.0  (0 = fixed length)

# ShareGPT dataset
--sharegpt-output-len Override output length

# Sonnet dataset
--sonnet-input-len    Default 550
--sonnet-output-len   Default 150
--sonnet-prefix-len   Default 200  (controls prefix-cache reuse)

# HF dataset
--hf-name --hf-subset --hf-split
```

### Canonical invocation

```bash
vllm bench serve \
  --model meta-llama/Llama-3.1-8B-Instruct \
  --backend vllm \
  --dataset-name sharegpt \
  --dataset-path ShareGPT_V3_unfiltered_cleaned_split.json \
  --num-prompts 1000 \
  --request-rate 10 \
  --save-result
```

Outputs include per-request TTFT, ITL, e2e latency, prompt/output token counts,
and aggregate p50/p95/p99 distributions.

---

## NVIDIA GenAI-Perf (and successor AIPerf)

GenAI-Perf is NVIDIA's **client-side** generative-AI benchmarking tool,
replacing `perf_analyzer` for LLMs. As of late 2025 NVIDIA positions
**AIPerf** (https://github.com/ai-dynamo/aiperf) as successor; CLI largely
compatible.

GenAI-Perf supports any OpenAI-compatible endpoint — works against vLLM,
SGLang, TGI, NIM, Triton+TRT-LLM uniformly.

### Canonical invocation

```bash
export INPUT_SEQUENCE_LENGTH=200
export INPUT_SEQUENCE_STD=10
export OUTPUT_SEQUENCE_LENGTH=200
export CONCURRENCY=10
export MODEL=meta/llama-3.1-8b-instruct

genai-perf profile \
    -m $MODEL \
    --endpoint-type chat \
    --service-kind openai \
    --streaming \
    -u localhost:8000 \
    --synthetic-input-tokens-mean $INPUT_SEQUENCE_LENGTH \
    --synthetic-input-tokens-stddev $INPUT_SEQUENCE_STD \
    --concurrency $CONCURRENCY \
    --output-tokens-mean $OUTPUT_SEQUENCE_LENGTH \
    --extra-inputs max_tokens:$OUTPUT_SEQUENCE_LENGTH \
    --extra-inputs min_tokens:$OUTPUT_SEQUENCE_LENGTH \
    --extra-inputs ignore_eos:true \
    --tokenizer meta-llama/Meta-Llama-3.1-8B-Instruct \
    -- -v --max-threads=256
```

### Recommended sweep loop

```bash
for concurrency in 1 2 5 10 50 100 250; do
   genai-perf profile ... --concurrency $concurrency \
     --measurement-interval 30000 \
     --profile-export-file ${ISL}_${OSL}.json
done
```

`--measurement-interval 30000` is critical: NVIDIA recommends "a value big
enough for several requests to finish... For larger networks, such as
Llama-3 70B, and more concurrency, such as 250, choose a higher value. For
example, you could use 100000 ms."

Outputs: `artifacts/<model>-<endpoint-type>-concurrency<N>/*.csv` and `*.json`.
"The `*genai_perf.csv` files are the main benchmarking results."

Tip: "We recommend starting a GenAI-Perf container on the same server as NIM
to avoid network latency, unless you specifically want to factor in the
network latency as part of the measurement."

---

## LLMPerf (Anyscale / ray-project)

Ray-based load tester. **GitHub repo archived Dec 17 2025**; methodology is
still cited heavily but new development has moved to other tools.

### Install
```bash
git clone https://github.com/ray-project/llmperf.git
cd llmperf && pip install -e .
```

### Canonical invocation
```bash
python token_benchmark_ray.py \
  --model "meta-llama/Llama-2-7b-chat-hf" \
  --mean-input-tokens 550 --stddev-input-tokens 150 \
  --mean-output-tokens 150 --stddev-output-tokens 10 \
  --max-num-completed-requests 2 \
  --timeout 600 \
  --num-concurrent-requests 1 \
  --results-dir result_outputs \
  --llm-api openai \
  --additional-sampling-params '{}'
```

### Methodology decisions (Anyscale blog)
- Always use the Llama-2 fast tokenizer to count tokens "in a system
  independent way", even when benchmarking GPT or Mistral.
  Reasoning: "the ChatGPT tokenizer is more 'efficient' than the Llama
  tokenizer (1.5 tokens per word for Llama 2 vs 1.33 tokens per word for
  ChatGPT). ChatGPT shouldn't be penalized for this."
- Default workload: **Mean input length 550 tokens (stdev 150); Mean output
  length 150 tokens (stdev 20)** — based on real Anyscale Endpoints traffic.
- Inputs/outputs assumed normally distributed (acknowledged limitation;
  Poisson would be more realistic).
- **TTFT is included in ITL** (different from GenAI-Perf).
- Standardize on **5 concurrent requests** as comparison anchor.

### Correctness test
`llm_correctness.py` sends number-conversion prompts ("one hundred and twenty
three" → 123) to verify accuracy doesn't degrade under load.

---

## NVIDIA Triton perf_analyzer

Pre-LLM workhorse; superseded by GenAI-Perf for generative workloads but
still relevant for embedding/rerank and non-streaming endpoints. perf_analyzer
measures throughput-vs-latency by sweeping concurrency or request rate, but
doesn't natively understand SSE streaming → can't compute TTFT/ITL on
streaming OpenAI endpoints. Use GenAI-Perf for any LLM with streaming.

---

## MLPerf Inference v5.0 — LLM track

MLCommons released v5.0 in April 2025, adding two LLM benchmarks important
for the multi-machine question.

### Llama 3.1 405B Instruct
- 128k-token context window benchmark (vs 4k for Llama 2 70B).
- Dataset: combined samples from LongBench, LongDataCollections, Ruler,
  GovReport-Summary. "Mean input/output context lengths of 9,400 and 680
  tokens respectively, our dataset stresses the model's ability to retain
  coherence and accuracy across extended sequences."
- Scenarios:
  - **Offline**: maximum tokens/sec, no latency cap.
  - **Server**: **99th-percentile TTFT ≤ 6 seconds** and
    **99th-percentile TPOT ≤ 175 ms**.
- Accuracy floor: "99% of the FP16 reference" (ROUGE-L for summarization,
  exact match for retrieval).
- Reference impl uses vLLM.

### Llama 2 70B Interactive
- Same model as v4.0, tightened SLOs.
- **99th-percentile TPOT ≤ 40 ms (25 tokens/sec/user)** and
  **99th-percentile TTFT ≤ 450 ms** — threshold MLCommons settled on after
  surveying ChatGPT and Perplexity in late 2024 and concluding "a 50th
  percentile token generation rate of 20–50 tokens per second (TPOT of
  20-50ms) is critical for seamless user experience."
- Dataset: OpenOrca; metric: ROUGE.

### Server scenario
Uses **open-loop Poisson arrival process** at a rate the submitter chooses;
submission is valid only if 99th-percentile latencies are met. Gold standard
for "publishing a number that means something."

### NVIDIA Blackwell numbers (GB200 NVL72, MLPerf v5.0)
"GB200 NVL72 delivers the highest Llama 3.1 405B performance per GPU,
showing 2.8x faster offline and 3.4x faster in server mode on the 405B
benchmark compared to H200."

---

## SGLang `bench_serving`

See `raw/sglang-distributed-2026-05.md` for the full flag set and methodology.

Key features for cross-engine work:
- Mirrors `vllm bench serve` design.
- Supports backends `sglang`, `sglang-oai`, `vllm`, `lmdeploy`, `trt`, `truss`
  — single harness against multiple engines.
- `generated-shared-prefix` dataset specifically exercises prefix-caching
  wins.
- `--flush-cache` for cold/warm cache comparisons.

---

## k6 / Locust patterns

k6 and Locust are not LLM-aware and produce misleading numbers if used
naively.

Specific traps:
- **k6** treats each request as a single unit; no native SSE/streaming
  awareness → no TTFT/ITL.
- **Locust** is single-process per worker; Python's GIL makes streaming-token
  tokenization compete with request generation → measurements biased high
  under load.

If you must use them:
- **k6**: streaming HTTP API; emit a custom event on each SSE chunk; subtract
  first-chunk timestamp from request start to get TTFT. Tag tokens (decoded
  client-side) and compute ITL between adjacent chunks.
- **Locust**: use `LLM Locust` (https://www.truefoundry.com/blog/llm-locust-a-tool-for-benchmarking-llm-performance)
  — fork that handles streaming and emits token-level metrics.
- Pair k6 with Grafana+InfluxDB so per-request distributions can be queried.

For published or capacity-sizing benchmarks, prefer GenAI-Perf, `vllm bench
serve`, or SGLang `bench_serving` over k6/Locust.

---

## Workload datasets / traces

### ShareGPT
`ShareGPT_V3_unfiltered_cleaned_split.json` — standard chat workload for
vLLM/SGLang. Real user-LLM dialogues; realistic length distributions.

Caveat: "the current implementation only uses the first 2 lines of each
dialog" — multi-turn fidelity lost. Don't use ShareGPT for multi-turn KV-cache
reuse measurement; use `generated-shared-prefix` (SGLang) or production-stack
multi-round-qa harness for that.

### LongBench
Multi-domain long-context benchmark, adopted by MLPerf v5.0 for the 405B
benchmark. Mean input ~9,400 tokens. Use it when measuring how the system
handles long-context prefill — TTFT scales roughly linearly with input
length.

### BurstGPT (Microsoft / HPC-AI Lab)
Most realistic public LLM trace.
- arXiv: https://arxiv.org/abs/2401.17644
- GitHub: https://github.com/HPMLL/BurstGPT
- **5.29M traces over 121 days** (BurstGPT_1, BurstGPT_2);
  **5.34M / 110 days** (BurstGPT_3); newer versions exceed 10M.
- Source: Azure OpenAI GPT-3.5 / GPT-4 service in single region.
- Schema: `Timestamp, Session ID, Elapsed time, Model, Request tokens,
  Response tokens, Total tokens, Log Type`.
- Captures: (1) concurrency patterns, (2) statistical relations between
  request and response lengths, (3) failed/throttled requests.

Recommended usage: "Scale the average Requests Per Second (RPS) in the trace
according to your evaluation setups" and "Model the patterns in the trace as
indicated in our paper and scale the parameters."

The trace to use for "what would a real production load look like" — not
synthetic Poisson arrivals at a fixed rate.

### Alpaca
Short-instruction prefill-heavy workload (~52k examples, ~150-token prompts,
~75-token outputs). Useful for prefill-bound benchmarking and tokenizer-stress
tests; not realistic for chat.

### vLLM "synthetic" / random workload
`--dataset-name random --random-input-len 1024 --random-output-len 128`
produces fixed-length synthetic prompts. Use:
- For apples-to-apples engine comparison (no tokenizer noise, no length var).
- For specific prefill/decode phase measurement (set ISL high or OSL high).
- **Not** for capacity sizing — real traffic is variable; synthetic understates
  p99.

### Multi-turn / agentic / tool-calling traces

**No widely-adopted public trace** for agentic tool-calling production
workloads. Closest options:

- **BFCL v3 / v4** — multi-turn function-calling traces; synthetic.
- **AgentBoard** — 9 tasks, 1013 environments, multi-turn agent traces
  (NeurIPS 2024).
- **MCP-Bench** — execution-trace-based metrics for MCP-style tool use.
- **MINT** — multi-turn tool interaction benchmark.

For **serving** benchmarks (not capability) on agentic traffic, the
practical approach:
1. Capture your own production traces with timestamps and tool-call
   boundaries.
2. Replay through `vllm bench serve --dataset-name custom` or
   `sglang.bench_serving --dataset-name generated-shared-prefix`.
3. Measure prefix-cache hit rate as a primary metric; agentic workloads
   should hit 70–90% if routing is correct.

Currently a gap in the public ecosystem — flag the absence of "BurstGPT for
agents."

---

## Benchmark traps / how to do it right

### Cold cache vs warm cache
- vLLM/SGLang/Dynamo all have automatic prefix caching.
- First 10–100 requests in any benchmark have abnormally high TTFT.
- A "best-of" measurement is a measurement of the cache, not the model.

Fix:
- Always **warm up** before measuring (`--num-warmups N` in `vllm bench
  serve`, or discard first N requests).
- For cold-cache (worst-case), use `--flush-cache` (SGLang) or restart server
  / vary prompt prefix.
- Report cache hit rate alongside latency.
- vLLM SqueezeBits study: "throughput reduction of ~36.7% and a TPOT
  increase of ~25.0%" when prefix caching is on but no prefixes are shared
  — "free" caching has worst-case overhead.
- For prefix-heavy workloads, cache hit rates can reach 87% with "88% faster
  time-to-first-token for warm cache hits" (target 80%+ for stable production
  prompts).

### Batch size 1 vs realistic concurrency
Batch-1 measurements characterize per-user TPOT floor; nothing about
production cost. A 14B model at batch-1 might do 60 tok/s; at batch-32 might
still do ~50 tok/s/user but ~1,600 tok/s aggregate — 27× cost difference.

Always sweep concurrency. NVIDIA's 1, 2, 5, 10, 50, 100, 250 sweep is the
convention. Plot the latency-vs-throughput Pareto front — operating point
is wherever your SLO line crosses it.

### Open-loop vs closed-loop load generation — **this matters**

**Closed-loop (LLMPerf default)**: N concurrent virtual users; each waits
for its request to complete before sending the next. Number in flight bounded
by N. Measures system at fixed concurrency. Real-world: matches "N user
sessions actively typing."

**Open-loop (MLPerf Server, GenAI-Perf with `--request-rate`)**: requests
arrive at target rate (often Poisson), regardless of whether server has
finished prior requests. Queue grows without bound; latency reveals
saturation. Real-world: matches "an external service hits us at fixed RPS."

**Why this matters**: a closed-loop test at N=100 will *automatically slow
down* if system saturates (because users wait), making the system look
"stable" at impossible loads. An open-loop test at the same effective rate
will show queues blowing up. **Use open-loop** when sizing for SLO; use
closed-loop when characterizing per-user experience at known concurrency.

LLMPerf is closed-loop. GenAI-Perf does both: `--concurrency N` is closed-loop;
`--request-rate R` is open-loop. MLPerf Server is strictly open-loop with
Poisson arrivals.

### Tokenizer skew
Different models use different tokenizers. Same English prompt produces
different token counts:
- ChatGPT/cl100k: ~1.33 tokens/word
- Llama 2 SentencePiece: ~1.5 tokens/word (~12% more)
- Llama 3 tiktoken-style: ~1.35 tokens/word

If you measure tokens/second using the model's own tokenizer, you penalize
less-efficient tokenizers. Anyscale: "we always use the Llama 2 fast
tokenizer to estimate the number of tokens in a system independent way."

For cost ($/Mtok) calculations, use the **model's own** tokenizer (it's what
the customer is billed on). For engine comparison, use a single standardized
tokenizer.

### GPU utilization is NOT a quality metric for inference
`nvidia-smi` "GPU-Util" is the percentage of time the kernel scheduler had
any work — it can show 100% during a tight memory-bandwidth-bound decode loop
that's leaving 90% of the SMs idle.

Correct utilization metrics:
- **MFU (Model Flops Utilization)**: measured TFLOPS / peak TFLOPS. Useful
  when compute-bound (large prefills).
- **MBU (Model Bandwidth Utilization)**: see definitions doc.

In typical decode-heavy serving, MBU is binding. A serving stack at **80%
MBU** is well-tuned regardless of `nvidia-smi`.

### Other traps
- **Streaming on/off matters.** Non-streaming responses don't have meaningful
  TTFT; many tools default to non-streaming. Always pass `--streaming`
  (GenAI-Perf) or check equivalent flag.
- **`ignore_eos: true` for fixed-length output benchmarks.** Otherwise
  different models terminate at different lengths and your "256 output
  tokens" is actually 256 ± 100.
- **Network locality.** Co-locate the load generator with the server unless
  measuring network. NVIDIA: "start a GenAI-Perf container on the same
  server as NIM to avoid network latency."
- **Measurement interval.** GenAI-Perf's `--measurement-interval` must be
  long enough for several requests to finish at every concurrency level — for
  405B at concurrency 250, NVIDIA recommends 100 seconds.
- **Warm-up.** First N requests load weights into cache, populate prefix
  cache, JIT-compile kernels. Discard or use `--num-warmups`.
- **Variance.** Run each configuration ≥ 3 times; report mean ± stdev.
  Cloud GPU variance is real (Databricks: 2× across providers for "the same"
  8×A100 box).
