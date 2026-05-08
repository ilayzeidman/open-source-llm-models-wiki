---
tags: [source, concepts, benchmarks, serving, performance]
source_path: raw/llm-serving-perf-metrics-2026-05.md
ingested: 2026-05-08
last_updated: 2026-05-08
---

# LLM serving performance metrics (May 2026)

Metric definitions used throughout the wiki — sourced from NVIDIA, Anyscale,
Databricks, vLLM design docs, the DistServe paper, llm-d, and MLPerf v5.0.

## Key claims

### TTFT — Time to First Token

NVIDIA: "the time it takes to process the prompt and generate the first
token", encompassing "request queuing time, prefill time, and network latency."
"the longer the prompt, the larger the TTFT."

GenAI-Perf and LLMPerf "disregard the initial responses that have no content
or content with an empty string."

Anyscale recommends reporting as a **distribution** (P50/P90/P95/P99).

vLLM: `vllm:time_to_first_token_seconds` (Histogram).

### ITL / TPOT — Inter-Token Latency / Time Per Output Token

NVIDIA: "average time between the generation of consecutive tokens."

**GenAI-Perf formula**: `ITL = (e2e_latency – TTFT) / (Total_output_tokens – 1)`
— excludes TTFT and the first token.

**Anyscale formula**: includes TTFT — "We've decided to include the TTFT in
the inter-token latency computation. We've seen some systems that start
streaming very late."

⚠ **Same workload can produce ITL numbers differing by 20–40% depending on
this convention.** Always check.

vLLM: `vllm:inter_token_latency_seconds` (Histogram).

### End-to-end latency

NVIDIA: `e2e_latency = TTFT + generation_time`.
Databricks: `latency = TTFT + TPOT × (number of tokens generated)`.
vLLM: `vllm:e2e_request_latency_seconds`.

### Throughput — TPS / RPS

- **System TPS**: total output tokens per second across all simultaneous
  requests.
- **Per-user TPS**: `output_seq_len / e2e_latency` — asymptotically approaches
  `1/ITL` for long outputs.
- **RPS**: completed requests per second.

LLMPerf bias warning (Anyscale): "in the single concurrency scenario can
sometimes account for 33% of the entire benchmark duration" of
non-inference overhead. GenAI-Perf excludes these.

### Goodput (DistServe — the headline multi-node metric)

- "the maximum request rate per second that the system can withhold while
  meeting a specified service level objective (SLO)."
- Conditioned on **percentile thresholds** + **multiple simultaneous latency
  constraints**, not aggregate measures.
- Concrete formulation: "Goodput (P90 TTFT < 200ms and P90 TPOT < 50ms) =
  maximum request rate per second when at least 90% of requests have both
  TTFT < 200ms and TPOT < 50ms."
- DistServe: "**2.0× – 3.41× higher goodput compared to vLLM**" (chatbot);
  3.2× (code); 4.48× (summarization).

### MBU — Model Bandwidth Utilization (Databricks)

- "MBU = (achieved memory bandwidth) / (peak memory bandwidth) where achieved
  = ((total model parameter size + KV cache size) / TPOT)."
- Worked example: 7B FP16 model + 14ms TPOT → 14 GB/14ms = 1 TB/s; on a
  2 TB/s GPU = 50% MBU.
- "Available and achieved memory bandwidth in inference hardware is a better
  predictor of speed of token generation than their peak compute performance."
- Decode-heavy serving target: ~80% MBU.

### Concurrency methodology — closed vs open loop

Critical distinction:

- **Closed-loop (LLMPerf default)**: N concurrent virtual users, each waits
  for response before sending next. Bounded in-flight count. Real-world:
  matches "N user sessions actively typing." **Auto-throttles under
  saturation** — masks overload.
- **Open-loop (MLPerf Server, GenAI-Perf `--request-rate`)**: requests
  arrive at fixed rate (often Poisson) regardless of server state. Queues
  grow. **Reveals saturation.** Real-world: matches "an external service
  hits us at fixed RPS."

Use **open-loop for SLO sizing**; closed-loop for per-user-experience
characterization at known concurrency.

### KV-cache hit rate metrics

vLLM Prometheus:
- `vllm:prefix_cache_queries` (Counter)
- `vllm:prefix_cache_hits` (Counter)
- `vllm:kv_cache_usage_perc` (Gauge)

PromQL hit rate: `rate(prefix_cache_hits_total[5m]) / rate(prefix_cache_queries_total[5m])`.

llm-d "Effective Cache Throughput" = "the number of prompt tokens per second
served directly from the cache." Right metric for KV-aware router evaluation.

llm-d 16-H100 cluster B2B workload result:
- Precise prefix-aware routing: TTFT P90 = **0.542 s**.
- Approximate routing: TTFT P90 = **31.083 s**.
- "**precise-scheduling is 57× faster than approximate-scheduling**."

Token cost ratio: cached tokens are 10× cheaper ($0.30 vs $3.00 per million).

### MLPerf Inference v5.0 SLOs (April 2025)

**Llama 3.1 405B Instruct** (long context, mean ISL/OSL 9,400/680):
- Server scenario: 99th-percentile TTFT ≤ **6 s**, 99p TPOT ≤ **175 ms**.
- Reference impl: vLLM.

**Llama 2 70B Interactive**:
- 99p TPOT ≤ **40 ms** (25 tok/s/user), 99p TTFT ≤ **450 ms**.
- Threshold rationale: "a 50th percentile token generation rate of 20–50
  tokens per second (TPOT of 20-50ms) is critical for seamless user
  experience" (MLCommons survey of ChatGPT and Perplexity, late 2024).

Server scenario uses **open-loop Poisson arrivals**.

NVIDIA Blackwell (GB200 NVL72): "highest Llama 3.1 405B performance per GPU,
showing 2.8× faster offline and 3.4× faster in server mode... compared to
H200."

### Real-hardware Dynamo numbers (SemiAnalysis InferenceMAX)

- GB200 NVL72 + Dynamo+vLLM: **12,587 tok/s/GPU** on Kimi K2.5 NVFP4 8k/1k.
- Single B200 node: **4,021 tok/s/GPU**.
- ~3× per-GPU win at scale from disaggregation.

### Citation URLs

- https://developer.nvidia.com/blog/llm-benchmarking-fundamental-concepts/
- https://developer.nvidia.com/blog/llm-inference-benchmarking-how-much-does-your-llm-inference-cost/
- https://www.anyscale.com/blog/reproducible-performance-metrics-for-llm-inference
- https://www.databricks.com/blog/llm-inference-performance-engineering-best-practices
- https://docs.vllm.ai/en/latest/design/metrics/
- https://arxiv.org/abs/2401.09670 (DistServe goodput)
- https://llm-d.ai/blog/kvcache-wins-you-can-see
- https://blog.vllm.ai/2025/01/27/v1-alpha-release.html
- https://huggingface.co/blog/tgi-benchmarking
- https://huggingface.co/blog/martinigoyanes/llm-inference-at-scale-with-tgi
- https://mlcommons.org/2025/04/llm-inference-v5/
- https://developer.nvidia.com/blog/nvidia-blackwell-delivers-massive-performance-leaps-in-mlperf-inference-v5-0/

## Pages updated on ingest

- [[concepts/serving-performance-measurement]] (NEW)
- [[concepts/disaggregated-serving]] (NEW)
- [[concepts/benchmarks]]
- [[infrastructure/vllm]]
- [[infrastructure/nvidia-dynamo]]
