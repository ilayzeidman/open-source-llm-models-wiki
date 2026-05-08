---
tags: [concepts, benchmarks, serving, performance, multi-node]
last_updated: 2026-05-08
source_count: 3
---

# Serving performance measurement

How to measure LLM serving performance honestly, especially across multiple
machines. The wiki distinguishes **capability benchmarks** (BFCL, SWE-bench,
LiveCodeBench — see [[concepts/benchmarks]]) from **serving benchmarks**
(this page) — what a deployed cluster can actually deliver under load.

The whole methodology rests on five rules:

1. **Measure goodput, not throughput** (under stated SLO).
2. **Use open-loop load generation when sizing for SLO.**
3. **Always report distributions** (p50/p95/p99), never just means.
4. **State the cache state** (cold / warm / cleared) and warm up first.
5. **Use one tokenizer** when comparing engines; the **model's own** for cost.

## Core metrics

[Source: [[sources/llm-serving-perf-metrics-2026-05]]]

### TTFT — Time to First Token

NVIDIA: "the time it takes to process the prompt and generate the first
token", encompassing "request queuing time, prefill time, and network
latency." "the longer the prompt, the larger the TTFT."

Dominated by prefill compute on the prompt; rises with input length and with
queue depth. Most user-perceived for streaming UIs.

vLLM Prometheus: `vllm:time_to_first_token_seconds` (Histogram).

### ITL / TPOT — Inter-Token Latency / Time Per Output Token

NVIDIA: "average time between the generation of consecutive tokens." Same
quantity, two names.

> ⚠ **Two formulas in the wild**:
>
> **GenAI-Perf**: `ITL = (e2e_latency – TTFT) / (Total_output_tokens – 1)`.
> Excludes TTFT. ITL = pure decode characteristic.
>
> **LLMPerf / Anyscale**: includes TTFT. "We've seen some systems that start
> streaming very late in the end-to-end-time."
>
> The same workload can produce ITL numbers differing by 20–40% depending on
> convention. Always check.

### End-to-end latency

NVIDIA: `e2e_latency = TTFT + generation_time`.
Databricks user-form: `latency = TTFT + TPOT × (number of tokens generated)`.

### Throughput — TPS / RPS

- **System TPS** = total output tokens per second across all requests.
- **Per-user TPS** = `output_seq_len / e2e_latency` — asymptotically `1/ITL`.
- **RPS** = completed requests per second.

### Goodput (DistServe — the headline multi-node metric)

[Source: [[sources/disaggregated-serving-papers-2026-05]]]

DistServe authors:

> "the maximum request rate per second that the system can withhold while
> meeting a specified service level objective (SLO)."

Conditioned on **percentile thresholds + multiple latency constraints**, not
aggregate measures.

Concrete formulation:

> "Goodput (P90 TTFT < 200ms and P90 TPOT < 50ms) = maximum request rate per
> second when at least 90% of requests have both TTFT < 200ms and TPOT <
> 50ms."

Motivating example: a system with "throughput of 10 requests per second.
But with the latency constraint, only 3 requests hold within the SLO
constraint, yielding a goodput of 3 requests per second... a user of this
high-throughput but low-goodput serving system will still suffer from low
quality of service."

**This is the metric to report for any "1 → 2 → 5 machine" scaling claim.**
Raw throughput at saturation is meaningless if the saturation point produces
TTFT > 6 s.

### MBU — Model Bandwidth Utilization (Databricks)

`MBU = ((model_params + KV_cache_size) / TPOT) / peak_memory_bandwidth`

Worked example: "if a 7B parameter running with 16-bit precision has TPOT
equal to 14ms, then it's moving 14GB of parameters in 14ms translating to
1TB/sec bandwidth usage. If the peak bandwidth of the machine is 2TB/sec,
we are running at an MBU of 50%."

**Decode-heavy serving target: ~80% MBU.**

Critical observation: "Available and achieved memory bandwidth in inference
hardware is a better predictor of speed of token generation than their peak
compute performance."

### KV-cache hit rate

vLLM Prometheus: `vllm:prefix_cache_queries`, `vllm:prefix_cache_hits`,
`vllm:kv_cache_usage_perc`.

PromQL: `rate(prefix_cache_hits_total[5m]) / rate(prefix_cache_queries_total[5m])`.

llm-d defines **Effective Cache Throughput** = "the number of prompt tokens
per second served directly from the cache."

llm-d 16-H100 cluster B2B workload: precise prefix-aware routing yields TTFT
P90 = **0.542 s** vs **31.083 s** for approximate routing — a **57× win**.
Token cost ratio: cached tokens are 10× cheaper.

## Benchmark tools

[Source: [[sources/llm-serving-perf-tools-2026-05]]]

| Tool | Mode | Streaming | Open / Closed loop | When to use |
|---|---|---|---|---|
| `vllm bench serve` | CLI in vLLM | yes | both | vLLM-native runs |
| **NVIDIA GenAI-Perf** | Standalone client | yes | both | Engine-agnostic; use against any OpenAI-compat endpoint |
| **AIPerf** (GenAI-Perf successor) | Standalone | yes | both | Same as GenAI-Perf, newer |
| **LLMPerf** (archived Dec 2025) | Ray-based | yes | closed | Methodology still cited; tool no longer maintained |
| **SGLang `bench_serving`** | Module | yes | both | SGLang-native; cross-engine via `--backend vllm` |
| **MLPerf Inference v5.0 Server** | Standardized | yes | open (Poisson) | Cross-organization comparison |
| **k6 / Locust** | Generic | poor | varies | Avoid for LLMs unless using LLM-aware fork |

### Recommended sweep
NVIDIA's canonical concurrency sweep: `1, 2, 5, 10, 50, 100, 250`. Plot
RPS vs TTFT to find the Pareto front; SLO line crosses it at the operating
point.

`--measurement-interval` must be long enough for several requests to finish
at every concurrency. NVIDIA: for Llama-3 70B at 250 concurrency, 100,000 ms.

### MLPerf Inference v5.0 SLOs (April 2025)

- **Llama 3.1 405B**: 99p TTFT ≤ **6 s**, 99p TPOT ≤ **175 ms** (mean ISL/OSL
  9,400/680).
- **Llama 2 70B Interactive**: 99p TPOT ≤ **40 ms** (25 tok/s/user), 99p
  TTFT ≤ **450 ms**.

These are reasonable SLO defaults if you have no preset target.

## Workload datasets

| Dataset | Use |
|---|---|
| **ShareGPT** (`ShareGPT_V3_unfiltered_cleaned_split.json`) | Default chat workload; realistic length distribution |
| **LongBench** | Long-context benchmarks (MLPerf 405B uses this) |
| **BurstGPT** (https://github.com/HPMLL/BurstGPT) | Most realistic public LLM trace; 5.29M+ requests over 121 days from Azure OpenAI; captures bursts |
| **Alpaca** | Short-instruction prefill-heavy |
| **vLLM/SGLang `random`** | Fixed-length synthetic; engine comparison only — **understates p99** |
| **`generated-shared-prefix`** (SGLang) | Designed to exercise prefix-cache wins |

⚠ **No widely-adopted public agentic-tool-call trace.** BFCL v3/v4,
AgentBoard, MCP-Bench, MINT exist but are synthetic. Production agentic
serving benchmarks require capturing your own traces and replaying via
`vllm bench serve --dataset-name custom`.

## Multi-node-specific issues

[Source: [[sources/llm-serving-perf-metrics-2026-05]]]

### Per-node vs cluster-wide throughput
- **Sum-of-replicas**: easy but misleading if replicas imbalanced.
- **Client-observed at the front-end**: the only number that matches user
  experience.

When publishing, always state (a) which model, (b) which router, (c) load-
generation point.

### Per-replica metrics for KV-aware routers
- Per-replica request count distribution (Gini coefficient or max/min
  ratio).
- Per-replica cache hit rate.
- Cross-replica session affinity rate.
- Inter-replica imbalance: p99 TTFT max-vs-min across replicas.

vLLM Production Stack Grafana dashboard tracks: "Available vLLM instances
(healthy count); Request latency distribution; TTFT distribution; Running
and pending requests per instance; GPU KV Usage Percent and GPU KV Cache Hit
Rate."

### Tail latency under load (queue time)
vLLM exposes `vllm:request_queue_time_seconds` — interval "between QUEUED
and most recent SCHEDULED" events.

Saturation pattern:
- TTFT p50 stays flat as RPS rises.
- TTFT p99 rises sharply once `running_requests` saturates KV budget.
- Queue time rises faster than total TTFT.

Plot p50 / p95 / p99 of TTFT *and* queue time on the same chart against
RPS. The knee point of p99 is the operating ceiling.

### Saturation rules of thumb

- **TGI vertical-vs-horizontal**: throughput-vs-latency scatter; "If your
  batch size is in a horizontal area, this means you are compute bound and
  increasing users just delays everyone with no benefit of throughput."
- **NVIDIA Pareto front**: sweep concurrency 1–250; find inflection.
- **Goodput collapse**: most rigorous indicator — find RPS at which goodput
  (under your SLO) stops growing.

## Cost — $ per million tokens (NVIDIA methodology)

1. Run GenAI-Perf concurrency sweep → latency–throughput Pareto front.
2. Identify latency constraint ("TTFT ≤ 250 ms").
3. Filter to configurations meeting constraint; pick highest-throughput.
4. Required model instances = `peak RPS / per-instance RPS at SLO`.
5. Yearly server cost = `(initial / depreciation) + hosting + licensing`.
6. Cost per 1,000 prompts = `yearly cost / yearly request volume`.
7. Apportion to input/output (output ≈ 3× input — most public APIs).

⚠ Per-token cost only makes sense **at a stated SLO**. Without SLO, $/Mtok
is not comparable.

### Anyscale: input vs output token impact
"each additional input token adds 0.3-0.7 ms to the end-to-end time, compared
to each output token which adds 30-60 ms... Thus input tokens have
approximately 1% of the impact of output tokens on end-to-end latency."

## Common traps

[Source: [[sources/llm-serving-perf-tools-2026-05]]]

| Trap | Fix |
|---|---|
| Cold cache vs warm cache | Warm up; use `--num-warmups`. SqueezeBits: prefix caching with no shared prefix can cost ~36.7% throughput. |
| Batch size 1 vs realistic concurrency | Always sweep concurrency. |
| **Open-loop vs closed-loop** | Use **open-loop** (Poisson) for SLO sizing; closed-loop misleadingly auto-throttles under saturation. |
| Tokenizer skew | Use one standardized tokenizer for engine comparison; model's own for $/Mtok. |
| GPU-Util ≠ throughput | `nvidia-smi` shows 100% during memory-bound decode that leaves SMs idle. Use MFU/MBU. |
| Streaming on/off | Always pass `--streaming`. |
| `ignore_eos: true` | For fixed-length output benchmarks. |
| Network locality | Co-locate client with server. |
| Variance | Run ≥ 3×; report mean ± stdev. Cloud GPU variance: 2× across providers (Databricks). |

## Open-loop vs closed-loop — why it matters

[Source: [[sources/llm-serving-perf-metrics-2026-05]]]

| | Closed-loop (LLMPerf default) | Open-loop (MLPerf Server, GenAI-Perf `--request-rate`) |
|---|---|---|
| Pattern | N concurrent users; each waits for response before sending next | Requests arrive at fixed rate (Poisson); queue grows |
| In-flight count | Bounded by N | Unbounded |
| Saturation behavior | **Auto-throttles** under load (users wait) | **Reveals saturation** (queues blow up) |
| Real-world match | "N users typing in a chat UI" | "External service hits us at fixed RPS" |
| When to use | Per-user-experience characterization at known concurrency | **SLO sizing** |

A closed-loop test at N=100 will *automatically slow down* if the system
saturates, making it look "stable" at impossible loads. **Use open-loop**
when sizing for SLO.

## A10G g5 — what to measure

For the wiki's "1 → 2 → 5 g5.xlarge" scaling story, the canonical recipe is:

1. **Establish baseline** on 1× g5.xlarge:
   - Run GenAI-Perf or `vllm bench serve` against ShareGPT, sweep concurrency.
   - Record TTFT p50/p95/p99, ITL p50/p95/p99, RPS at saturation.
   - State your SLO (suggest MLPerf Llama 2 Interactive: 99p TTFT ≤ 450 ms,
     99p TPOT ≤ 40 ms — adjust if your UX needs are different).
   - Compute goodput.
2. **Scale to 2 g5.xlarge** behind [[infrastructure/vllm-production-stack]]
   with prefix-aware routing.
   - Re-run the same sweep at the cluster front-end (open-loop, not
     sum-of-replicas).
   - Track per-replica imbalance, cache hit rate.
3. **Scale to 5 g5.xlarge.** Repeat. Watch for non-linear scaling — most
   commonly from imbalanced routing (one replica over-loaded).

See [[comparisons/scaling-1-to-5-machines]] for the full recipe with
specific commands and SLO budgets.

## Related

- [[concepts/benchmarks]] — capability benchmarks (BFCL, SWE-bench, LCB)
- [[concepts/parallelism-strategies]]
- [[concepts/disaggregated-serving]]
- [[infrastructure/vllm]]
- [[infrastructure/vllm-production-stack]]
- [[infrastructure/llm-d]]
- [[comparisons/scaling-1-to-5-machines]]

## Sources

- [[sources/llm-serving-perf-metrics-2026-05]]
- [[sources/llm-serving-perf-tools-2026-05]]
- [[sources/disaggregated-serving-papers-2026-05]]
