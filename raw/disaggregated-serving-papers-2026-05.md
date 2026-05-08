# Disaggregated serving — primary papers

Three papers form the canonical theory of prefill/decode disaggregation that
underlies NVIDIA Dynamo, vLLM PD-disagg, SGLang PD-disagg, and llm-d.

## DistServe (Zhong et al., OSDI 2024)

- arXiv: https://arxiv.org/abs/2401.09670
- USENIX OSDI '24: https://www.usenix.org/system/files/osdi24-zhong-yinmin.pdf
- UCSD Hao AI Lab blog: https://haoailab.com/blogs/distserve/

### Headline claim
"**Serve 7.4× more requests or 12.6× tighter SLO**" while maintaining latency
for >90% of requests.

### Problem
Existing systems suffer "strong prefill-decoding interferences" and couple
resource allocation; DistServe assigns prefill/decoding to different GPUs and
"co-optimizes the resource allocation and parallelism strategy tailored for
each phase."

### Goodput definition (introduced here)
- "the number of completed requests per second that adheres to SLOs (TTFT and
  TPOT requirements)"
- "the maximum request rate per second that the system can withhold while
  meeting a specified service level objective (SLO)."

Motivating example: a system with "a throughput of 10 requests per second.
But with the latency constraint, only 3 requests hold within the SLO
constraint, yielding a goodput of 3 requests per second... a user of this
high-throughput but low-goodput serving system will still suffer from low
quality of service."

Argument vs raw throughput: "Throughput measures the number of requests or
tokens completed across all users and requests, hence overlooking these
latency requirements."

Concrete formulation:
> **"Goodput (P90 TTFT < 200ms and P90 TPOT < 50ms) = maximum request rate per
> second when at least 90% of requests have both TTFT < 200ms and TPOT < 50ms."**

### Per-workload results
- **DistServe sustains 2.0x – 3.41x higher goodput compared to vLLM** for
  chatbots.
- "3.2x higher goodput and 1.5x more stringent SLO than vLLM" for code
  completion.
- "4.48x higher goodput and 10.2x more stringent SLO than vLLM" for
  summarization.

### Compute vs memory characterization
"Prefill is very compute-bound, meaning a small batch of prefills or even a
single long enough prefill will easily saturate GPU computation. On the other
hand, decoding needs a much bigger batch size to hit the compute bound, and
is more easily subject to the memory bandwidth limit of the GPU."

---

## Splitwise (Microsoft, ISCA 2024)

- arXiv: https://arxiv.org/abs/2311.18677

### Headline claim
"**1.4× higher throughput at 20% lower cost**; 2.35× throughput at same
cost/power."

### Problem framing
LLM inference has "a compute-intensive prompt computation, and a memory-
intensive token generation, each with distinct latency, throughput, memory,
and power characteristics."

### Hardware insight (cost lever)
"Token generation phases do not require the compute capability of the latest
GPUs, and can be run with lower power and cost."

I.e. you can run prefill on H100 and decode on cheaper memory-bandwidth-rich
GPUs (e.g., A100/L40S) for total cost reduction. This is the academic basis
for Dynamo's "heterogeneous prefill/decode" path.

---

## Mooncake (Moonshot AI, July 2024)

- arXiv: https://arxiv.org/abs/2407.00079

### Headline claim
"**Up to a 525% increase in throughput**" simulated; **75% more requests** in
production at Kimi.

### Architecture
"KVCache-centric disaggregated architecture that separates the prefill and
decoding clusters."

Adds "a prediction-based early rejection policy" for overload.

### Distinctive contribution
A third axis: **KV cache as a distributed resource**. Mooncake treats KV cache
as a first-class fabric across the cluster — prefill nodes write blocks to
shared storage, decoder nodes pull on demand. This is the conceptual seed for
Dynamo's KVBM and llm-d's KV-aware routing.

### Long-context strength
Particularly effective for long-context: KV cache reuse across requests with
large shared prefixes (e.g., RAG, agent traces) is the bulk of the
acceleration.

---

## Synthesis

All three converge on the same insight: prefill is **compute-bound**, decode
is **memory-bandwidth-bound**, and co-locating them causes interference and
forces over-provisioning. Disaggregation lets each phase use a tailored
parallelism strategy and scaling factor. Mooncake adds a third axis (KV cache
as a distributed resource).

NVIDIA's Dynamo is a productionization of these ideas; vLLM v1, SGLang, and
TRT-LLM all now support disaggregated serving as first-class citizens.

⚠ Numbers above are paper-reported on specific workloads — actual gains depend
on input/output ratios, model size, and SLO targets. Don't expect "7.4×" on
your workload without verifying.
