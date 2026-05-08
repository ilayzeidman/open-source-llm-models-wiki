---
tags: [source, infrastructure, kubernetes, multi-node]
source_path: raw/k8s-llm-orchestration-2026-05.md
ingested: 2026-05-08
last_updated: 2026-05-08
---

# Kubernetes LLM orchestration — LWS / llm-d / vLLM Production Stack / AIBrix

Building blocks for multi-replica and multi-node vLLM on Kubernetes.

## Key claims

### LeaderWorkerSet (LWS, kubernetes-sigs)

- "An API for deploying a group of pods as a unit of replication."
- Replica = one logical inference unit; scales as a unit.
- Each replica has 1 leader pod (separate template, exposes API) +
  N worker pods (shared template).
- Group-level rolling updates; optional gang scheduling; topology-aware
  placement.
- Env to pods: `LWS_LEADER_ADDRESS`, `LWS_GROUP_SIZE`, plus index 0…n-1.
- Critical setting: `restartPolicy: RecreateGroupOnPodRestart` — single
  worker failure restarts whole replica (Ray/NCCL state bound at process
  start).
- LWS itself doesn't handle network primitives — Ray (or `vllm serve
  --headless`) does the rendezvous.
- For EFA on EKS, request `vpc.amazonaws.com/efa: N` and `hugepages-2Mi`.
- GKE Llama-3.1-405B reference config: 2× a3-highgpu-8g (8× H100 each),
  TP=8 within node, PP=2 across nodes.

### llm-d

- K8s Operator built jointly with KServe community.
- Releases: 0.4 (2025), 0.5 (2026), 0.7 (Apr 2026 current).
- CRDs: `LLMInferenceService`, `LLMInferenceConfig`.
- Integrates with KServe "Inference Gateway Extension" (Envoy + Gateway API
  Inference Extension) for "prefix-cache aware routing."
- Endpoint Picker (EPP) + InferencePool CRD scores pods by "real-time
  metrics, KV-cache affinity, and configured policies."
- Reported gains: "3× improvement in output tokens/s" and "2× reduction in
  TTFT."
- KV-cache experiment (16-H100 cluster, B2B workload, 150 customers ×
  6,000-token contexts × 5 concurrent users):
  - TTFT P90 with **precise prefix-aware** routing: **0.542 s**.
  - TTFT P90 with **approximate** routing: **31.083 s**.
  - "**precise-scheduling is 57× faster than approximate-scheduling**."
- Token cost ratio: "tokens already in the cache is 10× lower than for
  uncached tokens ($0.30 vs $3.00 per million)."

### vLLM Production Stack

- Apache-2.0; released January 2025.
- "open-source reference implementation of an inference stack built on top
  of vLLM."
- Components: Serving Engine (vLLM pods), Request Router, Observability
  (Prometheus + Grafana).
- Routing algorithms: round robin; session-ID stickiness; **prefix-aware**;
  KV-cache-aware.
- Helm-deployable: `helm install vllm vllm/vllm-stack -f values.yaml`.
- Performance claim: "10× better performance with 3-10× lower response delay
  & 2-5× higher throughput" (vendor).
- Router v0.5 (Dec 2025): hierarchical KV offloading; cache-aware LoRA
  routing; active-active HA; scale-to-zero; UCCL transport resilience.
  Validated: ~3.1k tok/s/B200 decode (wide-EP); 50k output tok/s on 16×16
  B200 P/D topology.
- Router blog claim vs llm-d: "vLLM Router throughput is 25% higher than
  llm-d and 100% higher than K8s-native load balancer, and vLLM Router's TTFT
  is 2000 ms faster."
- Benchmark methodology: `benchmarks/multi-round-qa/` simulates N concurrent
  users × M rounds with shared system prompts. Per-replica imbalance and
  KV-cache hit rate must be scraped from Prometheus.

### AIBrix

- "open-source initiative... essential building blocks to construct scalable
  GenAI inference infrastructure."
- Components: LLM Gateway and Routing; High-Density LoRA Management;
  LLM-tailored autoscaler; distributed KV cache; **heterogeneous serving**
  (mixed GPU types); Unified AI Runtime sidecar; GPU hardware failure
  detection.
- Stack: Go + Python + TypeScript on K8s.

### KServe

- CNCF/Linux Foundation; Apache-2.0.
- Predictive + generative AI under one CRD model.
- LLM-specific surface: `LLMInferenceService` (introduced with llm-d).
- LLM runtimes: vLLM, llm-d, native HuggingFace.
- Features: scale-to-zero, autoscaling, canary, A/B, request-based
  autoscaling, KV-cache offload to CPU/disk.

### Citation URLs

- LWS: https://github.com/kubernetes-sigs/lws
- LWS docs: https://lws.sigs.k8s.io/docs/examples/vllm/
- vLLM LWS: https://docs.vllm.ai/en/stable/deployment/frameworks/lws/
- LWS sample manifest: https://raw.githubusercontent.com/kubernetes-sigs/lws/refs/heads/main/docs/examples/vllm/GPU/lws.yaml
- GKE multi-host serving: https://docs.cloud.google.com/kubernetes-engine/docs/tutorials/serve-multihost-gpu
- llm-d: https://llm-d.ai/, https://github.com/llm-d/llm-d
- llm-d KV blog: https://llm-d.ai/blog/kvcache-wins-you-can-see
- llm-d production blog: https://llm-d.ai/blog/production-grade-llm-inference-at-scale-kserve-llm-d-vllm
- llm-d architecture: https://llm-d.ai/docs/architecture
- vLLM Production Stack: https://github.com/vllm-project/production-stack
- Production Stack docs: https://docs.vllm.ai/en/latest/deployment/integrations/production-stack/
- KV-aware tutorial: https://docs.vllm.ai/projects/production-stack/en/vllm-stack-0.1.5/tutorials/kvaware.html
- Router release blog: https://blog.vllm.ai/2025/12/13/vllm-router-release.html
- AIBrix docs: https://aibrix.readthedocs.io/latest/
- AIBrix repo: https://github.com/aibrix/aibrix
- KServe: https://kserve.github.io/website/

## Pages updated on ingest

- [[infrastructure/leaderworkerset]] (NEW)
- [[infrastructure/llm-d]] (NEW)
- [[infrastructure/vllm-production-stack]] (NEW)
- [[infrastructure/aibrix]] (NEW)
- [[comparisons/scaling-1-to-5-machines]] (NEW)
