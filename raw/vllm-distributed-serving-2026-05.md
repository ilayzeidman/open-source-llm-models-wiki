# vLLM distributed serving — May 2026 docs sweep

Consolidated from official vLLM docs, GitHub READMEs, and published blogs.
Sources fetched 2026-05-08.

## Authoritative URLs

- Parallelism overview: https://docs.vllm.ai/en/stable/serving/parallelism_scaling/
- Data parallel deployment: https://docs.vllm.ai/en/latest/serving/data_parallel_deployment/
- Expert parallel deployment: https://docs.vllm.ai/en/latest/serving/expert_parallel_deployment/
- Distributed serving (v0.9.0): https://docs.vllm.ai/en/v0.9.0/serving/distributed_serving.html
- LWS deployment: https://docs.vllm.ai/en/stable/deployment/frameworks/lws/
- ROCm vLLM MoE Playbook: https://rocm.blogs.amd.com/software-tools-optimization/vllm-moe-guide/README.html
- Red Hat: Wide EP for DeepSeek with vLLM/llm-d:
  https://developers.redhat.com/articles/2025/09/08/scaling-deepseek-style-moes-vllm-and-llm-d-using-wide-ep
- vLLM V1 alpha blog (2025-01-27):
  https://blog.vllm.ai/2025/01/27/v1-alpha-release.html

## Decision tree (verbatim from docs.vllm.ai)

- "If your model fits in a single GPU, you probably don't need to use distributed inference."
- Single-node multi-GPU: TP "when your model is too large to fit in a single
  GPU, but it can fit in a single node with multiple GPUs."
- Multi-node: combine TP and PP "when your model is too large to fit in a
  single node."
- The canonical rule: "**The tensor parallel size should be the number of GPUs
  in each node, and the pipeline parallel size should be the number of nodes.**"
  Example: 16 GPUs across 2 nodes → `--tensor-parallel-size 8 --pipeline-parallel-size 2`.
- For uneven GPU counts within a node: "Enable pipeline parallelism, which
  splits the model along layers and supports uneven splits."
- For nodes without NVLink (e.g. L40S/A10G): "leverage pipeline parallelism
  instead of tensor parallelism for higher throughput and lower communication
  overhead."

## Core CLI flags

| Flag | Meaning |
|---|---|
| `--tensor-parallel-size N` | Shard each layer across N GPUs (intra-node, NVLink/PCIe) |
| `--pipeline-parallel-size N` | Split layers across N stages (across slow links / multi-node) |
| `--data-parallel-size N` | N independent model replicas |
| `--data-parallel-size-local N` | DP ranks on this node |
| `--data-parallel-address`, `--data-parallel-rpc-port` | Coordination endpoint |
| `--data-parallel-start-rank N` | Rank offset on worker nodes |
| `--data-parallel-backend ray` | Use Ray for DP launch |
| `--data-parallel-hybrid-lb` | Each node has its own API server |
| `--enable-expert-parallel` | EP for MoE experts |
| `--all2all-backend {allgather_reducescatter\|deepep_high_throughput\|deepep_low_latency\|flashinfer_nvlink_one_sided\|flashinfer_nvlink_two_sided}` | All-to-all kernel |
| `--enable-eplb` + `--eplb-config '{...}'` | Expert load balancer |
| `--distributed-executor-backend ray\|mp` | Ray vs multiprocessing |
| `--nnodes`, `--node-rank`, `--master-addr`, `--headless` | MP multi-node mode |
| `--api-server-count N` | Scale OAI API workers (avoids API bottleneck at large DP) |

## Multi-node recipes

### Ray cluster mode (16 GPU / 2 node)

```bash
# Head
bash run_cluster.sh vllm/vllm-openai <HEAD_NODE_IP> --head /path/to/hf -e VLLM_HOST_IP=<HEAD_NODE_IP>
# Worker
bash run_cluster.sh vllm/vllm-openai <HEAD_NODE_IP> --worker /path/to/hf -e VLLM_HOST_IP=<WORKER_NODE_IP>
# Serve
vllm serve /path/to/model \
  --tensor-parallel-size 8 --pipeline-parallel-size 2 \
  --distributed-executor-backend ray
```

### Multiprocessing mode (no Ray)

```bash
# Head (rank 0)
vllm serve /path/to/model --tensor-parallel-size 8 --pipeline-parallel-size 2 \
  --nnodes 2 --node-rank 0 --master-addr <HEAD_IP>
# Worker (rank 1)
vllm serve /path/to/model --tensor-parallel-size 8 --pipeline-parallel-size 2 \
  --nnodes 2 --node-rank 1 --master-addr <HEAD_IP> --headless
```

### Data parallel + EP for DeepSeek-class MoE on 2 H200/H100 nodes

```bash
# Node 1 primary
vllm serve deepseek-ai/DeepSeek-V3-0324 \
  --all2all-backend deepep_low_latency \
  --tensor-parallel-size 1 --enable-expert-parallel \
  --data-parallel-size 16 --data-parallel-size-local 8 \
  --data-parallel-address 192.168.1.100 --data-parallel-rpc-port 13345 \
  --api-server-count 8

# Node 2 secondary
vllm serve deepseek-ai/DeepSeek-V3-0324 \
  --all2all-backend deepep_low_latency \
  --tensor-parallel-size 1 --enable-expert-parallel \
  --data-parallel-size 16 --data-parallel-size-local 8 \
  --data-parallel-start-rank 8 \
  --data-parallel-address 192.168.1.100 --data-parallel-rpc-port 13345 \
  --headless
```

## TP vs PP vs DP vs EP — when to use each

| Strategy | Splits | Comm pattern | Best when |
|---|---|---|---|
| **TP** | Each layer's matmul across GPUs | AllReduce per layer | Model > 1 GPU; fast intra-node interconnect (NVLink/NVSwitch). Low TTFT/ITL. |
| **PP** | Layers across GPUs | P2P between adjacent stages | Cross-node or PCIe/L40S/A10G (no NVLink). Per-step transfer = `B·S·H·(P−1)·bytes` vs TP's `4·B·S·H·L·bytes` → "near order-of-magnitude reduction" (LMSYS). |
| **DP** | Replicas | None for dense; AllReduce dummy passes for MoE | Throughput; one replica fits a GPU/node. Independent: "If one GPU handles 50 req/s, adding a second with DP=2 gets you ~100." |
| **EP** | MoE experts across GPUs | All-to-all (DeepEP/PPLX) | Sparse MoE only. `EP_SIZE = TP_SIZE × DP_SIZE`. Required: `--enable-expert-parallel`. |

## MoE-specific guidance (vLLM EP docs + ROCm Playbook)

- DeepSeek with MLA: "**Always use EP=1 with DP to avoid 8× KV cache duplication.**"
- ROCm crossover (MoE): ≤128 concurrent → TP+EP wins; ≥512 → DP+EP wins.
  Crossover 256–512.
- Backend choice:
  - `deepep_high_throughput` (grouped GEMM) — prefill-heavy.
  - `deepep_low_latency` (CUDA graphs) — decode-heavy.
  - `flashinfer_nvlink_*` — cross-node NVLink (NVL72-class).
- DP coordinator: "When any requests are in progress in any rank, we must
  ensure that empty 'dummy' forward passes are performed in all ranks that
  don't currently have any requests scheduled."

## Communication backends

- **NCCL** (default) for collectives. Ref: https://github.com/NVIDIA/nccl.
  "optimized for various high-performance interconnects including PCIe, NVLink,
  NVswitch, InfiniBand Verbs, and TCP/IP sockets."
- **GLOO** for CPU-side coordination (less common for vLLM core path).
- **NIXL** for KV transfer in disaggregated paths (appears as `nixl` in EP
  communicator options).

## Verifying NCCL transport

```
NCCL_DEBUG=TRACE vllm serve ...
```
Look for `[send] via NET/IB/GDRDMA` (InfiniBand path); on AWS,
`NET/OFI Selected Provider is efa`.

## Multi-replica vs multi-node TP/PP — decision rubric

| Goal / Constraint | Scale by | Why |
|---|---|---|
| Model fits one GPU/node, need more QPS | **Replicas (DP / multi-replica)** | Zero inter-node traffic; embarrassing parallel; survives partial failures. |
| Model too big for one GPU but fits one node | **TP within node** | NVLink/NVSwitch tolerates AllReduce-per-layer cost; lowest TTFT/ITL. |
| Model too big for one node | **TP within node + PP across nodes** | Official vLLM rule. PP per-step transfer is `B·S·H·(P−1)·bytes` — orders less than TP. |
| Tight TTFT for single user (low concurrency) | **TP** | Splits compute, reduces wall-clock per token. |
| Maximize throughput for many concurrent | **DP replicas** (or DP+EP for MoE) | Independent batches; no sync tax. PP+TP also reaches higher peak throughput than PP+DP at same GPU count (~47.8% advantage at peak). |
| Disaggregate prefill (compute-bound) from decode (memory-bound) | **Disaggregated serving (Dynamo/DistServe-style)** | Each phase scales independently with right hardware. |
| Slow inter-node link (no NVLink, no EFA — i.e. **g5 on AWS**) | **PP across nodes**, replicas, or DP only — **avoid TP across nodes** | TP comm volume scales with `4·L`; PP is `(P-1)`. |
| MoE (DeepSeek-class) at scale | **DP attention + EP MoE** with `--enable-expert-parallel` | Avoid 8× MLA KV duplication. Crossover ~256–512 concurrency (TP+EP below, DP+EP above). |

## A10G g5 deployment guidance (specific to wiki primary target)

- **g5.xlarge** (1× A10G, 24 GB): single replica only. Scale by adding more
  g5 instances behind a router (vllm-router / production-stack / AIBrix) — i.e.
  multi-replica DP, **not** multi-node TP. EFA unavailable on g5; ENA only.
- Multi-instance scaling rule (1→2→5 machines): each is an independent
  replica; KV-cache-aware routing on the router for prefix-cache hits; only
  consider true multi-node serving if model exceeds 24 GB at picked quant
  (then move to g6e or use PP across g5s with caveat of slow ENA links).
- For a model that *barely* fits, prefer AWQ/GPTQ INT4 on a single g5 +
  replicas, over TP across two g5.xlarge.

## vLLM V1 engine (Jan 2025 release)

Architectural changes:
- `EngineCore` execution loop in separate process (multiprocessing).
- Unified scheduler "removes the traditional distinction between 'prefill' and
  'decode' phases" — schedule = `{request_id: num_tokens}`.
- Hash-based prefix caching with "less than 1% decrease in throughput even
  when the cache hit rate is 0%" — now on by default.
- Persistent batch with NumPy input prep.
- Symmetric TP architecture (scheduler decoupled from worker 0).
- Piecewise CUDA graphs + FlashAttention 3.

Headline: "V1 achieves up to 1.7x higher throughput compared to V0," "up to
24% improvement in throughput" on generation-heavy workloads (Red Hat blog
on 0.8.1).

How to enable: `export VLLM_USE_V1=1` (default in vLLM ≥ 0.8.x).

## vLLM Production Stack

See `raw/serving-stacks-alternatives-2026-05.md`.
