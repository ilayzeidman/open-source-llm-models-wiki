# Kubernetes LLM orchestration — May 2026 sweep

Covers LeaderWorkerSet, llm-d, vLLM Production Stack, AIBrix, KServe — the
Kubernetes building blocks for multi-replica and multi-node vLLM deployments.

## LeaderWorkerSet (LWS)

### URLs
- Project: https://github.com/kubernetes-sigs/lws
- vLLM example manifest: https://raw.githubusercontent.com/kubernetes-sigs/lws/refs/heads/main/docs/examples/vllm/GPU/lws.yaml
- vLLM LWS docs: https://docs.vllm.ai/en/stable/deployment/frameworks/lws/
- Docs site: https://lws.sigs.k8s.io/docs/examples/vllm/
- GKE tutorial (DeepSeek-R1 / Llama-3.1-405B):
  https://docs.cloud.google.com/kubernetes-engine/docs/tutorials/serve-multihost-gpu

### Definition
"**LeaderWorkerSet (LWS): An API for deploying a group of pods as a unit of
replication.**" Kubernetes SIG project. Addresses the gap between Deployment
(independent replicas) and StatefulSet (ordered identity, no leader/worker
template split).

### API model
- **Replica** = one logical inference unit; scales as a unit.
- **Leader pod** = single pod, separate template, exposes the API.
- **Worker pods** = N pods sharing a common template.
- Each pod gets a unique index 0…n-1 (env: `LWS_LEADER_ADDRESS`, `LWS_GROUP_SIZE`,
  plus index).
- Group-level rolling updates; optional gang scheduling; topology-aware
  placement.

### vLLM LWS reference manifest

```yaml
apiVersion: leaderworkerset.x-k8s.io/v1
kind: LeaderWorkerSet
metadata:
  name: vllm
spec:
  replicas: 2
  leaderWorkerTemplate:
    size: 2                    # 1 leader + 1 worker per replica
    restartPolicy: RecreateGroupOnPodRestart
    leaderTemplate:
      spec:
        containers:
        - name: vllm-leader
          image: docker.io/vllm/vllm-openai:latest
          command: ["sh","-c","bash /vllm-workspace/examples/online_serving/multi-node-serving.sh leader \
            --ray_cluster_size=$(LWS_GROUP_SIZE); \
            vllm serve meta-llama/Meta-Llama-3.1-405B-Instruct \
              --port 8080 --tensor-parallel-size 8 --pipeline_parallel_size 2"]
          resources:
            limits: { nvidia.com/gpu: "8", memory: 1124Gi, ephemeral-storage: 800Gi }
          ports: [{containerPort: 8080}]
          readinessProbe: { tcpSocket: { port: 8080 } }
          volumeMounts: [{ name: dshm, mountPath: /dev/shm }]
        volumes: [{ name: dshm, emptyDir: { medium: Memory, sizeLimit: 15Gi } }]
    workerTemplate:
      spec:
        containers:
        - name: vllm-worker
          image: docker.io/vllm/vllm-openai:latest
          command: ["sh","-c","bash /vllm-workspace/examples/online_serving/multi-node-serving.sh worker \
            --ray_address=$(LWS_LEADER_ADDRESS)"]
          resources:
            limits: { nvidia.com/gpu: "8", memory: 1124Gi, ephemeral-storage: 800Gi }
          volumeMounts: [{ name: dshm, mountPath: /dev/shm }]
        volumes: [{ name: dshm, emptyDir: { medium: Memory, sizeLimit: 15Gi } }]
---
apiVersion: v1
kind: Service
metadata: { name: vllm-leader }
spec:
  type: ClusterIP
  ports: [{ port: 8080 }]
  selector: { leaderworkerset.sigs.k8s.io/name: vllm, role: leader }
```

In this example: `size: 2` = 1 leader + 1 worker per replica; `replicas: 2` =
two independent inference groups, each spanning 2 nodes with 8 GPUs each
(16 GPUs/replica).

### GKE-confirmed configuration (Llama-3.1-405B)
- 2× `a3-highgpu-8g` nodes (8× H100 80 GB each).
- "**Two-way pipeline parallelism to shard the model across the two nodes**" +
  "**Eight-way tensor parallelism to shard the model across the eight GPUs on
  each node.**"
- Hyperdisk ML reduces 405B init from ~90 min → ~20 min.

### Gotchas
- LWS does not handle network primitives — Ray (or `vllm serve --headless`)
  does the actual rendezvous.
- For EFA on EKS, the LWS pod template must request `vpc.amazonaws.com/efa: N`
  and `hugepages-2Mi`.
- `RecreateGroupOnPodRestart` is critical: a single worker pod failure must
  restart the whole replica because Ray/NCCL state is bound at process start.

---

## llm-d

### URLs
- Site: https://llm-d.ai/
- GitHub: https://github.com/llm-d/llm-d
- Architecture: https://llm-d.ai/docs/architecture
- Production blog: https://llm-d.ai/blog/production-grade-llm-inference-at-scale-kserve-llm-d-vllm
- KV-cache wins: https://llm-d.ai/blog/kvcache-wins-you-can-see

### Description
Kubernetes Operator for production LLM serving on K8s; built jointly with the
KServe community.

### Releases
- 0.4 (2025), 0.5 (2026), 0.7 (Apr 2026, current).

### CRDs
- `LLMInferenceService`
- `LLMInferenceConfig`

### Integration
KServe "Inference Gateway Extension" (Envoy + Gateway API Inference Extension)
for "prefix-cache aware routing." Endpoint Picker (EPP) + InferencePool CRD —
"scores and selects model server pods based on real-time metrics, KV-cache
affinity, and configured policies."

### Reported gains
"3x improvement in output tokens/s" and "2x reduction in time to first token."

### KV-cache experimental result (16-H100 cluster, B2B workload)
- 150 customers × 6,000-token contexts × 5 concurrent users.
- TTFT P90 with **precise prefix-aware** routing: **0.542 s**.
- TTFT P90 with **approximate** routing: **31.083 s**.
- "precise-scheduling is 57x faster than approximate-scheduling."

### Token cost ratio
"The cost for processing tokens already in the cache is 10 times lower than
for uncached tokens ($0.30 vs. $3.00 per million)."

---

## vLLM Production Stack

### URLs
- GitHub: https://github.com/vllm-project/production-stack
- Docs: https://docs.vllm.ai/en/latest/deployment/integrations/production-stack/
- KV-aware tutorial: https://docs.vllm.ai/projects/production-stack/en/vllm-stack-0.1.5/tutorials/kvaware.html
- Router release blog: https://blog.vllm.ai/2025/12/13/vllm-router-release.html

### Tagline
"A reference implementation on how to build an inference stack on top of vLLM,
which allows you to scale from a single vLLM instance to a distributed vLLM
deployment without changing any application code."

### Components
1. **Serving Engine** — vLLM pods.
2. **Request Router** — routes by routing keys / session IDs to maximize KV
   cache reuse.
3. **Observability Stack** — Prometheus + Grafana.

### Routing algorithms
- Round robin
- Session-ID stickiness
- **Prefix-aware load balancing** — "ensures that subsequent requests with the
  same prompt prefix are routed to the same instance, maximizing KV cache
  utilization"
- KV-cache-aware routing — "smarter instance picks based on cached tokenized
  prompt"

### Deployment
- Helm: `helm install vllm vllm/vllm-stack -f values-…yaml`.
- "Routing to endpoints that run different models", "Automatic service
  discovery and fault tolerance via the Kubernetes API", model aliases.

### Performance claim
"10x better performance with 3-10x lower response delay & 2-5x higher
throughput" via prefix-aware routing + KV cache sharing across instances
(vendor-supplied).

### Router v0.5 (Dec 2025)
- Hierarchical KV offloading
- Cache-aware LoRA routing
- Active-active HA, scale-to-zero autoscaling
- UCCL-based transport resilience
- Validated: ~3.1k tok/s/B200 decode (wide-EP), 50k output tok/s on 16×16 B200
  P/D topology, "order-of-magnitude TTFT reduction vs round-robin baseline."

### Benchmark methodology
`benchmarks/multi-round-qa/` simulates **N concurrent users × M rounds per
user** with shared system prompts. Reports QPS, prompt throughput, generation
throughput, TTFT.

Limitation: doesn't directly measure KV-cache hit rate, prefix sharing, or
per-replica imbalance — these come from Prometheus metrics
(`vllm:prefix_cache_hits`, `vllm:kv_cache_usage_perc`) scraped during the run.

### vs llm-d (router release blog)
"vLLM Router throughput is 25% higher than llm-d and 100% higher than K8s-
native load balancer, and vLLM Router's TTFT is 2000 ms faster."

---

## AIBrix

### URLs
- Docs: https://aibrix.readthedocs.io/latest/
- GitHub: https://github.com/aibrix/aibrix

### Description
"An open-source initiative designed to provide essential building blocks to
construct scalable GenAI inference infrastructure." Cloud-native control
plane for vLLM clusters.

### Components
- **LLM Gateway and Routing** — multi-model, multi-replica.
- **High-Density LoRA Management.**
- **LLM App-Tailored Autoscaler** — scales on real-time demand.
- **Distributed KV Cache** — "high-capacity, cross-engine KV reuse."
- **Heterogeneous Serving** — "cost-effective SLO-driven LLM inference using
  heterogeneous GPUs."
- **Unified AI Runtime** — sidecar for metrics, model download,
  standardization.
- **GPU Hardware Failure Detection.**

### Stack
Go + Python + TypeScript; deploys via K8s.

---

## KServe

See `raw/serving-stacks-alternatives-2026-05.md` for the standalone summary.
Key fact for orchestration: KServe's `LLMInferenceService` CRD is the LLM-
specific surface introduced with the llm-d integration. KServe's role is the
unified inference platform; llm-d is the LLM-specific operator that plugs in.

---

## Decision rule (K8s-native LLM serving on AWS EKS)

- Single model, single GPU, prototype → **raw vLLM**.
- Multi-replica with prefix-aware routing on K8s → **vLLM Production Stack**.
- Full K8s operator with multi-model lifecycle, canary, autoscaling, GPU
  orchestration, predictive + generative under one CRD → **KServe + llm-d**
  (modern path) or **KServe + vLLM Serving Runtime** (simpler).
- Multi-node TP+PP single model (e.g. Llama-3.1-405B across 2× 8×H100 nodes)
  → **LWS** (the LeaderWorkerSet CRD), with Ray rendezvous inside.
- Heterogeneous GPU pool, high-density LoRA, distributed KV cache → **AIBrix**.
- Multi-cluster federation across regions → KServe + Karmada.
