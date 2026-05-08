---
tags: [infrastructure, serving, kubernetes, multi-node]
last_updated: 2026-05-08
source_count: 2
---

# LeaderWorkerSet (LWS)

Kubernetes API for deploying inference replicas as **pod groups** rather
than independent pods. The right K8s primitive when one logical inference
replica spans multiple pods (multi-host TP+PP, e.g. Llama-3.1-405B across
2× 8×H100 nodes). Apache-2.0; project under `kubernetes-sigs/lws`.
[Source: [[sources/k8s-llm-orchestration-2026-05]]]

## What it is

> "**LeaderWorkerSet (LWS): An API for deploying a group of pods as a unit
> of replication.**"

Addresses the gap between K8s `Deployment` (independent replicas) and
`StatefulSet` (ordered identity, but no leader/worker template split):

- **Replica** = one logical inference unit; scales as a unit.
- **Leader pod** = single pod, separate template, exposes API.
- **Worker pods** = N pods sharing common template.
- Each pod gets unique index 0…n-1.
- Env injected: `LWS_LEADER_ADDRESS`, `LWS_GROUP_SIZE`, plus index.

Group-level rolling updates; optional gang scheduling; topology-aware
placement.

## Reference manifest (vLLM Llama-3.1-405B across 2 nodes)

```yaml
apiVersion: leaderworkerset.x-k8s.io/v1
kind: LeaderWorkerSet
metadata:
  name: vllm
spec:
  replicas: 2                # two independent 405B serving groups
  leaderWorkerTemplate:
    size: 2                  # 1 leader + 1 worker per replica
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

In this example: `size: 2` = 1 leader + 1 worker per replica; `replicas: 2`
= two independent 405B serving groups, each spanning 2 nodes with 8 GPUs each
(16 GPUs/replica).

## GKE-confirmed reference (Llama-3.1-405B / DeepSeek-R1)

- 2× `a3-highgpu-8g` nodes (8× H100 80 GB each).
- "**Two-way pipeline parallelism to shard the model across the two nodes**"
  + "**Eight-way tensor parallelism to shard the model across the eight GPUs
  on each node.**"
- Hyperdisk ML reduces 405B init from ~90 min → ~20 min.

## Gotchas

- LWS does **not** handle network primitives — Ray (or `vllm serve --headless`)
  does the actual rendezvous between leader and workers.
- For EFA on EKS, the LWS pod template must request:
  ```yaml
  resources:
    limits:
      vpc.amazonaws.com/efa: 32     # for p5.48xlarge
      hugepages-2Mi: 5120Mi
  ```
- **`restartPolicy: RecreateGroupOnPodRestart` is critical**: a single worker
  pod failure must restart the whole replica because Ray/NCCL state is bound
  at process start.
- Choose `tensor-parallel-size` = GPUs per node; `pipeline-parallel-size` =
  number of nodes (the canonical vLLM rule).

## When to use LWS

- **One model so big it needs multiple nodes** (Llama-3.1-405B,
  DeepSeek-V3.1-671B, Qwen3-Coder-480B, Kimi-K2-1T).
- Each replica is multi-host but you want multiple replicas behind a router.
- You want K8s-native scheduling / rolling update for that group.

## When NOT to use LWS

- Each replica fits a single GPU/node → just `Deployment` of vLLM pods +
  Service, or [[infrastructure/vllm-production-stack]] for the router.
- You don't need K8s — multi-node serve via `vllm serve --nnodes ...
  --headless` works fine outside Kubernetes.

## A10G fit

✅ — but rarely needed at A10G class. Most A10G-fitting models (≤ 30B at AWQ
INT4) are single-replica per g5.xlarge, scaled by independent replicas, not
multi-host TP+PP. LWS becomes relevant when stepping up to 70B+ models that
require TP=4 or TP=8 across PCIe — the [[hardware/multi-gpu-options|multi-GPU
options page]] discusses when that crossover happens.

For this wiki's primary "1 → 2 → 5 g5.xlarge" question, LWS is the **wrong
tool** — use Production Stack instead. LWS shines once your model crosses
the single-node boundary.

## Related

- [[infrastructure/vllm]]
- [[infrastructure/sglang]]
- [[infrastructure/vllm-production-stack]]
- [[infrastructure/llm-d]]
- [[hardware/aws-efa]]
- [[concepts/parallelism-strategies]]
- [[comparisons/scaling-1-to-5-machines]]

## Sources

- [[sources/k8s-llm-orchestration-2026-05]]
- [[sources/vllm-distributed-serving-2026-05]]
