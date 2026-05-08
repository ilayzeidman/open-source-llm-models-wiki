---
tags: [source, infrastructure, vllm, multi-node]
source_path: raw/vllm-distributed-serving-2026-05.md
source_url: https://docs.vllm.ai/en/stable/serving/parallelism_scaling/
ingested: 2026-05-08
last_updated: 2026-05-08
---

# vLLM distributed serving (May 2026)

Authoritative parallelism guidance from docs.vllm.ai for multi-GPU and
multi-node vLLM deployments. Covers TP / PP / DP / EP, recipes, and
A10G-specific guidance.

## Key claims

### Decision tree (verbatim from docs.vllm.ai)

- "If your model fits in a single GPU, you probably don't need to use
  distributed inference."
- Single-node multi-GPU: TP "when your model is too large to fit in a single
  GPU, but it can fit in a single node with multiple GPUs."
- Multi-node: combine TP and PP "when your model is too large to fit in a
  single node."
- Canonical rule: **"The tensor parallel size should be the number of GPUs
  in each node, and the pipeline parallel size should be the number of
  nodes."**
- For uneven GPU counts within a node: "Enable pipeline parallelism, which
  splits the model along layers and supports uneven splits."
- For nodes without NVLink (A10G/L40S): "leverage pipeline parallelism instead
  of tensor parallelism for higher throughput and lower communication
  overhead."

### TP / PP / DP / EP cheat sheet

| Strategy | Splits | Comm pattern | Best when |
|---|---|---|---|
| **TP** | Each layer's matmul | AllReduce per layer | Model > 1 GPU; fast intra-node interconnect |
| **PP** | Layers | P2P between adjacent stages | Cross-node or PCIe-only |
| **DP** | Replicas | None (dense); dummy passes for MoE | Throughput; replica fits a GPU/node |
| **EP** | MoE experts | All-to-all (DeepEP/PPLX) | Sparse MoE only |

### Comm volume comparison (key for slow links)
- TP: `4·B·S·H·L·bytes` per layer
- PP: `B·S·H·(P−1)·bytes` per step
- "near order-of-magnitude reduction" with PP across nodes (LMSYS).

### Multi-node CLI flags

- `--tensor-parallel-size N`, `--pipeline-parallel-size N`, `--data-parallel-size N`
- `--data-parallel-size-local N`, `--data-parallel-start-rank N`
- `--data-parallel-address`, `--data-parallel-rpc-port`
- `--data-parallel-backend ray`, `--data-parallel-hybrid-lb`
- `--enable-expert-parallel`, `--all2all-backend ...`
- `--distributed-executor-backend ray|mp`
- `--nnodes`, `--node-rank`, `--master-addr`, `--headless`
- `--api-server-count N` (for large DP)

### MoE-specific

- DeepSeek with MLA: "Always use EP=1 with DP to avoid 8× KV cache
  duplication."
- ROCm crossover: ≤128 concurrent → TP+EP wins; ≥512 → DP+EP wins.
- Backend choice:
  - `deepep_high_throughput` — prefill-heavy
  - `deepep_low_latency` — decode-heavy
  - `flashinfer_nvlink_*` — cross-node NVLink

### A10G g5 deployment guidance

- **g5 does NOT support EFA.** Multi-node TP across g5 instances is not viable.
- For 1 → 2 → 5 g5 machines, scale by **independent replicas behind a
  router**, not by multi-node TP.
- For models that don't fit on one g5 even at AWQ INT4, use **PP across
  g5 instances** (PP comm volume tolerates ENA's 25–100 Gbps); accept the
  latency hit, or move to g6e.

### vLLM V1 engine (Jan 2025 release)

- `EngineCore` execution loop in separate process.
- Unified scheduler removes prefill/decode distinction.
- Hash-based prefix caching on by default; "less than 1% decrease in throughput
  even when the cache hit rate is 0%."
- Persistent batch with NumPy input prep.
- "V1 achieves up to 1.7× higher throughput compared to V0."
- Default in vLLM ≥ 0.8.x; enable explicitly with `VLLM_USE_V1=1`.

### Citation URLs

- https://docs.vllm.ai/en/stable/serving/parallelism_scaling/
- https://docs.vllm.ai/en/latest/serving/data_parallel_deployment/
- https://docs.vllm.ai/en/latest/serving/expert_parallel_deployment/
- https://docs.vllm.ai/en/v0.9.0/serving/distributed_serving.html
- https://docs.vllm.ai/en/stable/deployment/frameworks/lws/
- https://blog.vllm.ai/2025/01/27/v1-alpha-release.html
- https://rocm.blogs.amd.com/software-tools-optimization/vllm-moe-guide/README.html
- https://developers.redhat.com/articles/2025/09/08/scaling-deepseek-style-moes-vllm-and-llm-d-using-wide-ep

## Pages updated on ingest

- [[infrastructure/vllm]] — added multi-node section
- [[concepts/parallelism-strategies]] (NEW)
- [[comparisons/scaling-1-to-5-machines]] (NEW)
- [[hardware/multi-gpu-options]]
