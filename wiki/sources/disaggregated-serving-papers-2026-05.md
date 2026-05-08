---
tags: [source, paper, disaggregated, serving]
source_path: raw/disaggregated-serving-papers-2026-05.md
ingested: 2026-05-08
last_updated: 2026-05-08
---

# Disaggregated serving papers — DistServe / Splitwise / Mooncake

The three foundational papers underlying NVIDIA Dynamo, vLLM PD-disagg,
SGLang PD-disagg, and llm-d. All argue: prefill is compute-bound, decode is
memory-bandwidth-bound, and co-locating them creates interference and forces
over-provisioning.

## Key claims

### DistServe (Zhong et al., OSDI 2024)

- Headline: "**Serve 7.4× more requests or 12.6× tighter SLO**" while
  maintaining latency for >90% of requests.
- **Goodput** introduced here: "the number of completed requests per second
  that adheres to SLOs (TTFT and TPOT requirements)" — "the maximum request
  rate per second that the system can withhold while meeting a specified SLO."
- Concrete formulation: "Goodput (P90 TTFT < 200ms and P90 TPOT < 50ms) =
  maximum request rate per second when at least 90% of requests have both
  TTFT < 200ms and TPOT < 50ms."
- Per-workload results vs vLLM:
  - Chatbot: **2.0× – 3.41× higher goodput**.
  - Code completion: 3.2× higher goodput, 1.5× tighter SLO.
  - Summarization: **4.48× higher goodput**, 10.2× tighter SLO.
- Compute/memory characterization: "Prefill is very compute-bound...
  decoding... is more easily subject to the memory bandwidth limit of the GPU."

### Splitwise (Microsoft, ISCA 2024)

- Headline: "**1.4× higher throughput at 20% lower cost**; 2.35× throughput
  at same cost/power."
- Hardware insight: "Token generation phases do not require the compute
  capability of the latest GPUs, and can be run with lower power and cost."
- This is the academic basis for Dynamo's heterogeneous prefill/decode path
  (e.g., H100 prefill + L40S decode).

### Mooncake (Moonshot AI, July 2024)

- Headline: "**Up to a 525% increase in throughput**" simulated;
  **75% more requests** in production at Kimi.
- Architecture: "KVCache-centric disaggregated architecture that separates
  the prefill and decoding clusters." Adds "prediction-based early rejection
  policy."
- Distinctive contribution: KV cache treated as a first-class distributed
  fabric. Conceptual seed for Dynamo KVBM and llm-d KV-aware routing.
- Particularly effective for long-context workloads with large shared
  prefixes (RAG, agent traces).

### Citation URLs

- DistServe arXiv: https://arxiv.org/abs/2401.09670
- DistServe USENIX OSDI '24: https://www.usenix.org/system/files/osdi24-zhong-yinmin.pdf
- DistServe blog (UCSD Hao AI Lab): https://haoailab.com/blogs/distserve/
- Splitwise: https://arxiv.org/abs/2311.18677
- Mooncake: https://arxiv.org/abs/2407.00079

⚠ Numbers above are paper-reported on specific workloads (model size, ISL/OSL
ratio, SLO targets). Don't expect "7.4×" on your workload without verifying.

## Pages updated on ingest

- [[concepts/disaggregated-serving]] (NEW)
- [[concepts/serving-performance-measurement]] (NEW)
- [[infrastructure/nvidia-dynamo]]
- [[wiki/overview]]
