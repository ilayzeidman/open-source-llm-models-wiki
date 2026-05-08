# LLM serving performance metrics — definitions

Consolidated from NVIDIA Developer Blog, Anyscale, Databricks, vLLM design
docs, DistServe paper, llm-d, MLPerf v5.0. Sources fetched 2026-05-08.

## Authoritative URLs

- NVIDIA "LLM Inference Benchmarking: Fundamental Concepts":
  https://developer.nvidia.com/blog/llm-benchmarking-fundamental-concepts/
- NVIDIA "How Much Does Your LLM Inference Cost?":
  https://developer.nvidia.com/blog/llm-inference-benchmarking-how-much-does-your-llm-inference-cost/
- Anyscale "Reproducible Performance Metrics for LLM Inference":
  https://www.anyscale.com/blog/reproducible-performance-metrics-for-llm-inference
- Databricks "LLM Inference Performance Engineering: Best Practices":
  https://www.databricks.com/blog/llm-inference-performance-engineering-best-practices
- vLLM design metrics: https://docs.vllm.ai/en/latest/design/metrics/
- DistServe paper (goodput): https://arxiv.org/abs/2401.09670
- llm-d "KV-Cache Wins You Can See":
  https://llm-d.ai/blog/kvcache-wins-you-can-see
- vLLM V1 alpha: https://blog.vllm.ai/2025/01/27/v1-alpha-release.html
- TGI benchmarking (HF blog): https://huggingface.co/blog/tgi-benchmarking
- HF "LLM Inference at Scale with TGI":
  https://huggingface.co/blog/martinigoyanes/llm-inference-at-scale-with-tgi
- HF multi-backend: https://huggingface.co/blog/tgi-multi-backend

---

## Time to First Token (TTFT)

NVIDIA: "the time it takes to process the prompt and generate the first
token", encompassing "request queuing time, prefill time, and network
latency." "the longer the prompt, the larger the TTFT" because attention must
process the entire input sequence to populate the KV cache before generation
can begin.

GenAI-Perf and LLMPerf "disregard the initial responses that have no content
or content with an empty string (no token present)" when computing TTFT.

Databricks: "How quickly users start seeing the model's output after entering
their query."

Anyscale: "In streaming applications, the TTFT is how long before the LLM
returns the first token." Recommends reporting TTFT as a distribution
(P50, P90, P95, P99), not just a mean.

vLLM Prometheus: `vllm:time_to_first_token_seconds` (Histogram), measured
"relative to when the request was first received by the frontend."

---

## Inter-Token Latency (ITL) / Time Per Output Token (TPOT)

NVIDIA: "Intertoken latency (ITL) is the average time between the generation
of consecutive tokens in a sequence. It is also known as time per output
token (TPOT)."

### GenAI-Perf formula
```
ITL = (e2e_latency – TTFT) / (Total_output_tokens – 1)
```
NVIDIA: "GenAI-Perf does not include TTFT in the average calculation (as
opposed to LLMPerf, which does include the TTFT). The equation used for this
metric does not include the first token (hence subtracting 1 in the
denominator). This is done so that ITL is a characteristic of the decoding
part of the request processing only."

### Anyscale's stance (different)
"We've decided to include the TTFT in the inter-token latency computation.
We've seen some systems that start streaming very late in the end-to-end-time."
Anyscale's ITL is conservatively higher when TTFT spikes — useful for SLO
enforcement, less useful for isolating decode efficiency.

### Databricks
"Time to generate an output token for each user that is querying our system."
Example: "a TPOT of 100 milliseconds/tok would be 10 tokens per second per
user, or ~450 words per minute, which is faster than a typical person can
read."

### vLLM
`vllm:inter_token_latency_seconds` (Histogram).

⚠ When comparing vLLM, SGLang, GenAI-Perf, and LLMPerf numbers across blogs,
**always check whether ITL includes TTFT** — same workload, ITL numbers can
differ by 20–40%.

---

## End-to-end latency

NVIDIA: time "from submitting a query to receiving the full response, including
the time for queueing and batching and network latencies":
```
e2e_latency = TTFT + generation_time
```

Databricks user-facing form:
```
latency = TTFT + TPOT * (number of tokens generated)
```

vLLM: `vllm:e2e_request_latency_seconds`.

---

## Throughput: tokens/sec, requests/sec

NVIDIA distinguishes:
- **System TPS**: "total output tokens per seconds throughput, accounting for
  all the requests happening simultaneously" = `Total_output_tokens / (Ty – Tx)`
  (Tx = first start, Ty = last response end).
- **Per-user TPS**: "Output sequence length / e2e_latency" — "asymptotically
  approaches 1/ITL as the output sequence length increases."

Requests per second (RPS): "the average number of requests that can be
successfully completed by the system in a 1-second period" =
`total_completed_requests / (Ty – Tx)`.

### Anyscale TPS bias warning
LLMPerf TPS = "total output tokens divided by the entire benchmark duration,
which also includes the following overheads: Input prompt generation, Request
preparation, Storing the responses" — and "in the single concurrency scenario
can sometimes account for 33% of the entire benchmark duration." GenAI-Perf
excludes these.

### TGI's "vertical vs horizontal" rule
"If your batch size is in a vertical area, this is great, you can get more
throughput and handle more users for free. If your batch size is in a
horizontal area, this means you are compute bound and increasing users just
delays everyone with no benefit of throughput."

Databricks: "higher throughput typically requires processing multiple requests
concurrently, which increases individual TPOT."

---

## Goodput (DistServe)

The single most under-appreciated metric for distributed LLM serving.

DistServe authors (UCSD Hao AI Lab, OSDI '24): "the number of completed
requests per second that adheres to SLOs (TTFT and TPOT requirements)";
"the maximum request rate per second that the system can withhold while
meeting a specified service level objective (SLO)."

Motivating example: a system with "a throughput of 10 requests per second.
But with the latency constraint, only 3 requests hold within the SLO
constraint, yielding a goodput of 3 requests per second... a user of this
high-throughput but low-goodput serving system will still suffer from low
quality of service."

Argument vs throughput: "Throughput measures the number of requests or tokens
completed across all users and requests, hence overlooking these latency
requirements."

Concrete formulation:
> **"Goodput (P90 TTFT < 200ms and P90 TPOT < 50ms) = maximum request rate per
> second when at least 90% of requests have both TTFT < 200ms and TPOT < 50ms."**

DistServe results: "DistServe sustains 2.0x – 3.41x higher goodput compared
to vLLM" (chatbots), "3.2x" (code), "4.48x" (summarization).

---

## Prefill vs decode throughput

Databricks: prefill is parallel over input tokens; decode is sequential.
"Each generated token is appended to the input and fed back into the model to
generate the next token."

DistServe: "Prefill is very compute-bound, meaning a small batch of prefills
or even a single long enough prefill will easily saturate GPU computation. On
the other hand, decoding needs a much bigger batch size to hit the compute
bound, and is more easily subject to the memory bandwidth limit of the GPU."

Why disaggregated systems (Dynamo, DistServe, Splitwise, llm-d) report prefill
and decode separately:
- **Prefill TPS** = sum of input tokens consumed / time, dominated by prefill
  GPU pool.
- **Decode TPS** = sum of output tokens emitted / time, dominated by decode
  pool.

vLLM: `vllm:request_prefill_time_seconds`, `vllm:request_decode_time_seconds`.
NVIDIA AIPerf and GenAI-Perf emit per request.

Databricks: "every doubling of batch size just increases the latency without
increasing throughput" beyond compute-bound regime — sign decode pool at MFU
saturation.

---

## Concurrency

NVIDIA: "Concurrency is the number of concurrent users, each having one
active request, or equivalently the number of requests concurrently being
served by an LLM service."

### Critical methodological difference between tools
- **GenAI-Perf**: "always ensures N active requests throughout the
  benchmarking period" (true closed-loop with replenishment).
- **LLMPerf**: "sends out requests in batches of N requests, but there is a
  draining period where it waits for all the requests to complete before
  sending out the next batch" (round-based; under-utilizes between rounds).

Anyscale: "this may be slightly conservative since we completed the concurrent
requests in 'rounds' rather than as continuous querying."

When a benchmark reports "100 concurrent requests", you must specify which
model.

---

## p50/p95/p99 latencies; SLO/SLA definitions

DistServe: SLOs are "a set of targets that an LLM serving system must meet";
goodput is **conditioned on percentile thresholds and multiple simultaneous
latency constraints rather than aggregate measures.**

P90/P99 (not mean) is the right operating metric because:
1. Continuous batching means decode-step jitter from incoming prefills causes
   long tails.
2. KV-cache eviction under memory pressure causes occasional re-prefill,
   spiking TTFT.
3. Routing decisions in multi-node setups create cache-miss tail spikes.

MLPerf Inference v5.0 enforces **99th percentile** thresholds for both TTFT
and TPOT in the Server scenario.

Anyscale: always report at least P50, P90, P95, P99 for TTFT and ITL.

---

## Cost: $ per 1M input / output tokens

NVIDIA's canonical methodology:

1. Run a **GenAI-Perf** sweep across concurrency to get a latency–throughput
   Pareto front.
2. "Identify your latency constraint" (e.g., "keep the average time to first
   token at or below 250 ms").
3. Filter the chart to configurations meeting the constraint; pick the one
   with the highest throughput.
4. Compute required model instances = `peak RPS / per-instance RPS at SLO`.
5. **Yearly server cost** = `(initial server cost / depreciation years) +
   annual hosting + annual licensing`.
6. **Cost per 1,000 prompts** = `yearly server cost / yearly request volume`.
7. Apportion to input vs output tokens — most providers price output ≈ 3×
   input (e.g., "$1 per million input tokens and $3 per million output
   tokens") since output tokens drive the decode-bound cost.

Per-token cost only makes sense **at a stated SLO** — without that, $/Mtok
numbers are not comparable.

### Anyscale: input vs output token latency cost
"each additional input token adds 0.3-0.7ms to the end-to-end time, compared
to each output token which adds 30-60 ms... Thus input tokens have
approximately 1% of the impact of output tokens on end-to-end latency."

---

## MBU (Model Bandwidth Utilization) — Databricks

> "MBU is defined as (achieved memory bandwidth) / (peak memory bandwidth)
> where achieved memory bandwidth is ((total model parameter size + KV cache
> size) / TPOT)."

Worked example: "if a 7B parameter running with 16-bit precision has TPOT
equal to 14ms, then it's moving 14GB of parameters in 14ms translating to 1
TB/sec bandwidth usage. If the peak bandwidth of the machine is 2TB/sec, we
are running at an MBU of 50%."

Critical observation: "Available and achieved memory bandwidth in inference
hardware is a better predictor of speed of token generation than their peak
compute performance." Memory bandwidth, not TFLOPs, is the binding constraint
for batch-1 decode.

In typical decode-heavy serving, MBU is the binding metric. A serving stack
at **80% MBU** is well-tuned regardless of what `nvidia-smi` says.

---

## Tensor-parallel scaling data (Databricks, Llama-2-70B)

- "going from 4x to 8x GPUs only decreases latency by 0.7x at small batch
  sizes" — diminishing returns from communication overhead.
- "At batch size 16, latency with 4x is 33% lower than with 2x" — TP scales
  better at higher batches.

### Cloud-provider variance warning
"a 2x latency difference between 8xA100 servers from two cloud providers."
**Always benchmark on your actual deployment target.**

---

## KV-cache hit rate metrics

vLLM exposes:
- `vllm:prefix_cache_queries` (Counter)
- `vllm:prefix_cache_hits` (Counter)
- `vllm:kv_cache_usage_perc` (Gauge, 0–1)

vLLM design doc: "Record the queries and hits as counters rather than
pre-computed hit rate to enable user-defined PromQL calculations." PromQL:
```
rate(vllm:prefix_cache_hits_total[5m]) / rate(vllm:prefix_cache_queries_total[5m])
```

llm-d "Effective Cache Throughput" = "the number of prompt tokens per second
served directly from the cache. This metric quantifies the computational work
the GPUs avoided." This is the right metric for evaluating a KV-aware router.

llm-d 16-H100 cluster, B2B workload (150 customers × 6,000-token contexts ×
5 concurrent users):
- TTFT P90 with **precise prefix-aware** routing: **0.542 s**.
- TTFT P90 with **approximate** routing: **31.083 s**.
- "precise-scheduling is 57x faster than approximate-scheduling."

Token cost ratio: "The cost for processing tokens already in the cache is 10
times lower than for uncached tokens ($0.30 vs. $3.00 per million)."

---

## Multi-node-specific measurement issues

### Per-node vs cluster-wide throughput aggregation
- **Sum-of-replicas**: `cluster_TPS = Σ per-replica_TPS`. Easy but misleading
  if replicas imbalanced.
- **Client-observed**: drive load from a single client at the cluster
  front-end and let the router fan out. Only number that matches user
  experience.

When publishing multi-node throughput, always state (a) which model, (b)
which router, (c) load-generation point.

### Per-replica metrics for KV-aware routers
- **Per-replica request count distribution** — Gini coefficient or max/min
  ratio. Good routers should keep load balanced within ~20%.
- **Per-replica cache hit rate** — high variance across replicas means
  cache-aware routing is fighting load balance.
- **Cross-replica session affinity rate** — for session-ID routing, fraction
  of requests going to the same replica as the session's previous request.
- **Inter-replica imbalance (tail latency divergence)** — p99 TTFT max-vs-min
  across replicas.

vLLM Production Stack Grafana dashboard tracks: "Available vLLM instances
(healthy count); Request latency distribution; TTFT distribution; Running
and pending requests per instance; GPU KV Usage Percent and GPU KV Cache Hit
Rate."

### Tail latency under load (queue time)
vLLM exposes `vllm:request_queue_time_seconds` — interval "between QUEUED and
most recent SCHEDULED" events. Separates queueing from prefill and decode
latency.

Saturation pattern:
- TTFT p50 stays flat as RPS rises.
- TTFT p99 rises sharply once `running_requests` saturates KV budget.
- Queue time rises faster than total TTFT — sign requests are spending more
  time waiting than computing.

Recommendation: plot p50 / p95 / p99 of TTFT *and* queue time on the same
chart against RPS. The knee point of p99 is your operating ceiling.

### Saturation detection rules of thumb
- TGI heuristic: vertical vs horizontal area on throughput-vs-latency scatter.
- NVIDIA Pareto front: sweep concurrency 1, 2, 5, 10, 50, 100, 250 and plot
  RPS vs TTFT.
- **Goodput collapse**: most rigorous indicator — find RPS at which goodput
  (under your SLO) stops growing. Past that, throughput may rise but goodput
  does not.

---

## NVIDIA Dynamo benchmark numbers (real hardware)

SemiAnalysis InferenceMAX (late 2025):
- GB200 NVL72 with Dynamo+vLLM: **12,587 tok/s/GPU** on Kimi K2.5 NVFP4 8k/1k.
- Single B200 node: **4,021 tok/s/GPU**.
- ~3× per-GPU win from disaggregation at scale.

This is proof that "per-node" benchmarks fundamentally understate disaggregated
systems.
