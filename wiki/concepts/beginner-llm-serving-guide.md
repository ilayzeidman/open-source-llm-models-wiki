---
tags: [concepts, guide, beginner, learning-path]
last_updated: 2026-05-08
source_count: 0
---

# Beginner's guide to LLM serving on AWS

A learning path for someone who has never deployed an LLM before but wants
to understand the wiki end-to-end. Each step builds on the previous one.
Read top-to-bottom, do not skip ahead — every later page assumes the
vocabulary from the earlier ones. By the end you should be able to read
[[comparisons/scaling-1-to-5-machines]] and follow every line of it.

This page is a **synthesis / map** — it does not introduce new claims.
Numbers and details live on the linked pages.

## What problem are we even solving?

You have a trained open-source LLM (a big file of weights). You want to
**serve** it: accept HTTP requests, generate text, return tokens fast and
cheap. That is a different job from training.

```
   Training (someone else did this)         Serving (your job)
   ──────────────────────────────           ──────────────────
   datasets                                 HTTP request
       │                                        │
       ▼                                        ▼
   ┌─────────┐                             ┌────────────┐
   │ Trainer │  weeks, $$$$$               │   Engine   │  ms, $/Mtok
   │ on H100s│                             │  on GPU(s) │
   └────┬────┘                             └─────┬──────┘
        │ produces                               │ produces
        ▼                                        ▼
   model weights ─────────────────────►   stream of tokens
   (e.g., 60 GB file)                     "Hello, world..."
```

The wiki's primary target hardware is a single **NVIDIA A10G** GPU on an
AWS **g5.xlarge** box (24 GB VRAM, ~$1/hr). Everything in this guide is
oriented around: "how does a beginner reason about serving on this box,
and what changes when we want more boxes?" See [[overview]].

### A typical request, in one diagram

```
client ──HTTP──► serving engine ──► GPU(s) ──► tokens stream back
                       │
                       │ two phases on the GPU:
                       │
                       ▼
       ┌────────────────────────────────────────┐
       │   PREFILL                  DECODE      │
       │   "read the prompt"        "write the  │
       │   one big matmul of         answer"    │
       │   the whole prompt          one token  │
       │                             at a time  │
       │   compute-bound             memory-    │
       │                             bandwidth- │
       │                             bound      │
       └────────────────────────────────────────┘
```

Hold on to that prefill-vs-decode split — it shows up everywhere later.

---

# Step 1 — Foundations

Three pages. After this step, you should be able to answer:

1. "Why can't I just run TP across two g5.xlarge boxes?"
2. "Why is goodput a better number than throughput?"

If you cannot answer those two questions, do not move on.

## 1.1 Parallelism — how a model is split across GPUs

Read: [[concepts/parallelism-strategies]].

When a model is too big or too slow on one GPU, you split the work. There
are **four ways** to split. A beginner only needs to deeply understand
the first two; the others are good to recognize by name.

```
  ┌──────────── ONE FORWARD PASS THROUGH A MODEL ────────────┐
  │                                                           │
  │    layer 1 ─► layer 2 ─► layer 3 ─► ... ─► layer N        │
  │                                                           │
  └───────────────────────────────────────────────────────────┘
```

### Tensor Parallel (TP) — split each layer across GPUs

Each layer's matrix multiply is sliced; every GPU does part of the math
and they sum results after every layer.

```
                      One layer's matmul, split TP=2:

               GPU A                           GPU B
        ┌───────────────┐               ┌───────────────┐
        │ left half of  │               │ right half of │
        │ the matrix    │               │ the matrix    │
        └───────┬───────┘               └───────┬───────┘
                │                               │
                └──── AllReduce (sum) ──────────┘
                         (every layer!)
```

- Pros: lowest latency per token when the link between GPUs is fast
  (NVLink / NVSwitch).
- Cons: enormous network traffic. Once per layer × every request.

### Pipeline Parallel (PP) — split layers across GPUs

GPU A holds layers 1–N/2, GPU B holds layers N/2+1–N. A request flows
through them in series.

```
   request ─► GPU A (layers 1..N/2) ─► GPU B (layers N/2+1..N) ─► output
                       │                       ▲
                       └─── one P2P send ──────┘
                            (only at the boundary)
```

- Pros: tiny network traffic compared to TP (~`4·L`× less).
- Cons: latency for one request is higher; throughput needs many in
  flight at once to keep both stages busy.

### Data Parallel (DP) — independent replicas

Run two complete copies of the model. A router sends each request to
whichever copy is free.

```
              ┌──► replica 1 (full model)
   router ────┤
              └──► replica 2 (full model)
```

- Pros: zero inter-GPU traffic per request. Easiest to scale.
- Cons: each replica needs enough VRAM for the whole model.

### Expert Parallel (EP) — only for MoE models

Mixture-of-Experts models have many "experts" inside each layer; only
some run per token. EP scatters experts across GPUs. Beginners can skip
this until they specifically deploy an MoE model.

### The one rule to memorize

> **TP within a node, PP across nodes.** [Source: vLLM docs, see
> [[concepts/parallelism-strategies]]]

```
   node 1 (8 GPUs, NVLink)           node 2 (8 GPUs, NVLink)
   ┌─────────────────────┐           ┌─────────────────────┐
   │ GPU GPU GPU GPU     │           │ GPU GPU GPU GPU     │
   │ GPU GPU GPU GPU     │           │ GPU GPU GPU GPU     │
   │   TP=8 inside       │           │   TP=8 inside       │
   └──────────┬──────────┘           └──────────┬──────────┘
              │                                 │
              └────────── PP=2 across ──────────┘
                          (slow link)
```

## 1.2 AWS EFA — why the network decides what's possible

Read: [[hardware/aws-efa]].

EFA (Elastic Fabric Adapter) is AWS's special low-latency network device.
Without it, GPUs on different machines talk over normal TCP/ENA — way
too slow for tensor parallel.

```
                      Bandwidth between two boxes
                      ────────────────────────────

   g5.xlarge ◄──── ENA, 10–25 Gbps ────► g5.xlarge        ❌ NO EFA
                  (normal Ethernet)

   g6e.x      ◄──── EFA ────────────────► g6e.x           ✅ EFA
   p4d        ◄──── EFA, 400 Gbps ───────► p4d            ✅ EFA
   p5         ◄──── EFA, 3,200 Gbps ─────► p5             ✅✅ EFA v2
   p6-b300    ◄──── EFA, 6,400 Gbps ─────► p6-b300        ✅✅✅ peak
```

### Why this matters: TP traffic vs PP traffic

```
        TP across nodes                       PP across nodes
        ───────────────                       ───────────────
   AllReduce on every layer            One small send per stage
   (huge volume, every request)        boundary

   Needs >>>100 Gbps to be sane.       Tolerates 25 Gbps just fine.
```

So the **single most important fact for the wiki**:

```
   ┌──────────────────────────────────────────────────────────┐
   │  g5.xlarge has NO EFA.                                   │
   │  ⇒ You cannot run TP across two g5 boxes.                │
   │  ⇒ Multiple g5 boxes → independent replicas + a router.  │
   └──────────────────────────────────────────────────────────┘
```

Memorize this; it determines every recommendation later.

## 1.3 Measuring performance honestly

Read: [[concepts/serving-performance-measurement]].

Most beginner mistakes come from quoting the wrong number. Four metrics
matter; learn all four.

### TTFT — Time To First Token

```
   client sends prompt ───────────►
                                                    "H"
                                                    ▲
   ◄────────────── TTFT ───────────────────────────►│
                                                    │
                                                    └─ first token
                                                       streams back
```

How long the user waits before *anything* appears. Dominated by prefill.

### ITL / TPOT — Inter-Token Latency / Time Per Output Token

```
                "H"   "ello"   ","   " world"   "!"
                 │      │       │      │         │
                 └──ITL─┘       └──ITL─┘
```

The gap between consecutive tokens during streaming. Dominated by decode
(memory bandwidth). Asymptotically `1/ITL` = per-user tokens/sec.

### Throughput vs Goodput — the trap

A naive throughput number says "the server did 10 RPS." But how many of
those requests actually met the latency promise (the SLO)?

```
         Throughput says: "I served 10 requests this second."
                                ┌────────────────────┐
        10 requests come in:    │ ✓ ✓ ✗ ✓ ✗ ✓ ✗ ✓ ✓ ✓ │
                                │   3 of them were   │
                                │   too slow!        │
                                └────────────────────┘

         Goodput says: "I served 7 requests within the SLO."
                       This is the only number a user cares about.
```

> Goodput = max RPS where ≥ X% of requests still meet the latency target.
> Without an SLO attached, throughput is decoration. [Source: DistServe,
> see [[concepts/disaggregated-serving]]]

### Open-loop vs closed-loop load testing

```
   Closed-loop (LLMPerf default):
        ┌─ user 1 sends → waits → reads → sends → waits → reads ─┐
        ├─ user 2 ...                                             │
        └─ ... N parallel "users"                                 │
   Auto-throttles when the system is overloaded — looks fine even
   when it's actually saturated. ⚠

   Open-loop (MLPerf, GenAI-Perf --request-rate):
        request   request   request    request
        ────►     ────►     ────►      ────►       (Poisson arrivals)
                 Queue grows when system can't keep up.
   Reveals saturation. ✓ Use this for SLO sizing.
```

### Step 1 checkpoint

You should be able to say:

- "We can't TP two g5.xlarge because there's no EFA, only ENA, and TP's
  per-layer AllReduce would saturate ENA."
- "I want to know **goodput at TTFT_p99 ≤ 450 ms and TPOT_p99 ≤ 40 ms**,
  not just 'throughput.'"

If both feel natural, move on.

---

# Step 2 — The serving-stack landscape

Read in this order:

1. [[infrastructure/serving-stack-landscape]] (skim — don't memorize)
2. [[infrastructure/triton-vs-dynamo]]
3. [[infrastructure/sglang]]
4. [[infrastructure/lmdeploy]]
5. [[infrastructure/tensorrt-llm]] + [[infrastructure/tgi]] (quick read)

Goal: be able to pick an engine for a given workload, not memorize every
detail.

## 2.1 The menu

```
   ┌──────────────── ENGINES (do the actual generation) ────────────┐
   │                                                                 │
   │  vLLM ────► default; broad model support, many tool parsers     │
   │  SGLang ──► best for prefix-heavy / agentic / RAG (RadixAttn)   │
   │  LMDeploy ► best AWQ INT4 — fits big models on small GPUs       │
   │  TensorRT-LLM ► NVIDIA peak; FP8 / NVFP4 native                 │
   │  TGI ─────► HuggingFace; ⚠ in maintenance mode (avoid for new) │
   │  llama.cpp ► CPU / Apple Silicon / edge — out of scope here     │
   │                                                                 │
   └─────────────────────────────────────────────────────────────────┘

   ┌────── ORCHESTRATORS (sit above engines, on Kubernetes) ────────┐
   │                                                                 │
   │  vLLM Production Stack ► Helm; lightest path                    │
   │  llm-d ──────────────► Red-Hat; full KServe operator            │
   │  AIBrix ─────────────► heterogeneous GPU pools, dense LoRA      │
   │  Ray Serve LLM ─────► Pythonic compositions, multi-LoRA         │
   │  LeaderWorkerSet ───► one model spanning multiple nodes         │
   │  NVIDIA Dynamo ─────► PD-disaggregation + KV-aware routing      │
   │                                                                 │
   └─────────────────────────────────────────────────────────────────┘
```

You don't need them all. Pick by workload.

## 2.2 Triton vs Dynamo — clearing the naming mess

Read: [[infrastructure/triton-vs-dynamo]].

NVIDIA renamed things, and this confuses everyone:

```
   Old name (pre-2025):    New name (2025+):           What it is:
   ──────────────────      ─────────────────           ───────────
   Triton Inference         Dynamo-Triton              general inference
   Server                                              server (TF/ONNX/
                                                       Python/...)

   (didn't exist)           NVIDIA Dynamo              new LLM-specific
                                                       orchestrator
                                                       (PD-disagg etc.)
```

When someone says "Triton" today they may mean either. Always check the
year of the doc.

## 2.3 SGLang — the strongest vLLM alternative

Read: [[infrastructure/sglang]].

SGLang's secret weapon is **RadixAttention** — it caches KV across
requests using a tree (radix-trie) keyed by prefix tokens.

```
   Three requests share a long system prompt:

   "You are a helpful agent... [tool schema]... [user Q1]"
   "You are a helpful agent... [tool schema]... [user Q2]"
   "You are a helpful agent... [tool schema]... [user Q3]"
   ▲────────── shared prefix ───────────▲

   Without prefix cache:  prefill all 3, full cost each.
   With RadixAttention:   prefill once, reuse for the next two.

                  Radix-trie of cached KV:

                       (root)
                          │
                    "You are a..."     ◄─── shared
                          │
                  "[tool schema]..."   ◄─── shared
                       /  |  \
                     Q1   Q2   Q3      ◄─── per request
```

**Pick SGLang when** your traffic shares prefixes — agents, RAG,
multi-turn chat, big system prompts.

## 2.4 LMDeploy — the AWQ INT4 specialist

Read: [[infrastructure/lmdeploy]].

LMDeploy was built around fitting bigger models on smaller GPUs by
combining: AWQ-W4A16 + KV-cache quantization + prefix caching, all at
once.

```
   FP16 weights (default)         AWQ-W4A16 (LMDeploy)
   ────────────────────           ────────────────────
   1 weight = 16 bits             1 weight = 4 bits
   32B model ≈ 64 GB              32B model ≈ 16 GB
   ❌ won't fit A10G              ✅ fits A10G with room
```

**Pick LMDeploy when** you specifically need to squeeze a bigger model
onto a smaller GPU at INT4.

## 2.5 TensorRT-LLM and TGI

- **TensorRT-LLM** — NVIDIA's peak-performance engine; hardest to set
  up; best when you need NVFP4/MXFP4 native (Blackwell/Hopper) and
  every last token/sec.
- **TGI** — HuggingFace's older engine; **in maintenance mode** as of
  2026. HF officially recommends migrating to vLLM or SGLang. Recognize
  the name; don't pick it for new deployments.

### Step 2 checkpoint

Given a workload, you can say which engine fits:

```
   workload                                     pick
   ────────                                     ────
   default / "I just need to serve a model"    vLLM
   agents / RAG / shared system prompts        SGLang
   need to fit 32B+ on a 24 GB GPU             LMDeploy (AWQ INT4)
   peak NVIDIA throughput, FP8/FP4 native      TensorRT-LLM
   new project on Hopper/Blackwell             vLLM or SGLang or TRT-LLM
   anything new                                NOT TGI
```

---

# Step 3 — Scaling theory: disaggregated serving

Read: [[concepts/disaggregated-serving]], then re-read
[[infrastructure/nvidia-dynamo]] with this context.

Recall the prefill-vs-decode split from the very first diagram. The key
insight is that they have **opposite** hardware needs:

```
   Phase    Bottleneck             Wants:
   ─────    ──────────             ──────
   Prefill  compute (FLOPs)        peak-compute GPU, e.g. H100
   Decode   memory bandwidth       wide-memory GPU, e.g. L40S
```

When both phases live on one GPU, they fight each other:

```
   Co-located (the old way):

      ┌─ GPU ────────────────────────────────────┐
      │                                          │
      │  prefill running ──────► uses all SMs    │
      │                                          │
      │  decode waiting ──────► starved          │
      │                                          │
      └──────────────────────────────────────────┘
```

Disaggregated serving puts them on separate GPU pools, with the KV cache
shipped between them:

```
   Disaggregated (DistServe / Splitwise / Mooncake):

   ┌─ Prefill pool ───┐                     ┌─ Decode pool ──────┐
   │                  │   KV cache transfer │                    │
   │ H100 / B200      │ ────────────────►   │ L40S / A10G / ...  │
   │ peak compute     │                     │ wide memory bw     │
   │                  │                     │                    │
   └──────────────────┘                     └────────────────────┘

         "small batch, big matmuls"        "huge batch, decode tokens"
```

Productionization map (one-liner):

| Theory paper | Production system |
|---|---|
| DistServe (introduces *goodput*) | Dynamo, vLLM PD-disagg, SGLang PD-disagg |
| Splitwise (heterogeneous prefill/decode hardware) | Dynamo Planner, AIBrix |
| Mooncake (KV as a distributed resource) | Dynamo KVBM/NIXL, llm-d |

### When does disaggregation pay off?

```
   ✅ helps when                            ❌ does not help when
   ──────────────                           ─────────────────────
   long prefills (RAG, agents, big tool     short uniform prompts
   schemas, long contexts)                  (plain chat)

   tight TPOT SLO at high concurrency       single replica on a single
                                            GPU (Dynamo's own README:
                                            "your inference engine alone
                                            is probably sufficient")

   heterogeneous GPU pool available         no EFA between nodes — KV
                                            transfer becomes the new
                                            bottleneck (g5 on AWS)
```

For this wiki's primary 1 → 5 g5.xlarge story, disaggregation is **not**
the answer (no EFA). It becomes interesting only on g6e+ instances.

---

# Step 4 — Kubernetes orchestration

Once you have more than one machine, you need a thing in front of them
that knows where to send each request. Pick **one** path based on the
team's existing setup; don't try to learn all of them.

```
                       Sitting above the engines
                       ─────────────────────────

   simplest ────────────────────────────────────────────► most powerful

   vLLM Prod Stack    llm-d           AIBrix         Dynamo
   (Helm chart)       (KServe op-     (heterogeneous  (PD-disagg)
                       erator, can-    GPUs, dense
                       ary, scale-     LoRA)
                       to-zero)

   LeaderWorkerSet — orthogonal: only when one model spans multiple
   nodes (e.g., Llama-3.1-405B on 2× 8×H100).
```

### 4a — vLLM Production Stack (start here)

Read: [[infrastructure/vllm-production-stack]].

This is the lightest path — a Helm chart published by the vLLM team that
gives you replicas + a KV-aware router + observability. If you're new to
K8s LLM orchestration, this is where you start.

```
   client ──HTTP──► vllm-router ──► replica 1 (vLLM on g5.xlarge)
                       │ │
                       │ └─prefix-aware─► replica 2 (vLLM on g5.xlarge)
                       │
                       └────────────────► replica N

   Prometheus scrapes each replica:
     - vllm:time_to_first_token_seconds
     - vllm:prefix_cache_hits_total
     - vllm:request_queue_time_seconds
```

### 4b — llm-d (KServe path)

Read: [[infrastructure/llm-d]]. Pick if you're already on KServe and
want a full operator (canary, multi-model, mixed predictive +
generative ML).

llm-d's headline measurement is what to remember: precise prefix-aware
routing yields **TTFT P90 = 0.542 s vs 31.083 s** for approximate
routing — a 57× win. That's why "router that knows about prefix caches"
matters at all.

### 4c — LeaderWorkerSet (multi-node single replica)

Read: [[infrastructure/leaderworkerset]].

Different shape: not "many replicas of a small model" but "one big model
spread across multiple nodes."

```
   one logical replica spanning two nodes:

   ┌── leader pod ─────────┐    ┌── worker pod ─────────┐
   │ g6e.48xlarge 8× L40S  │    │ g6e.48xlarge 8× L40S  │
   │ TP=8 inside           │    │ TP=8 inside           │
   └──────────┬────────────┘    └──────────┬────────────┘
              └── PP=2 across (over EFA) ──┘
```

Skim aibrix and ray-serve-llm for awareness only:
[[infrastructure/aibrix]], [[infrastructure/ray-serve-llm]].

---

# Step 5 — Synthesis: scaling 1 → 2 → 5 machines

Read end-to-end: [[comparisons/scaling-1-to-5-machines]] and
[[comparisons/serving-stacks-comparison]].

Now everything lands as one concrete recipe:

```
   ┌── 1 g5.xlarge ──┐
   │                 │
   │  client ──► vLLM (one process, one GPU)
   │                 │
   │  baseline: GenAI-Perf concurrency sweep, record
   │            TTFT/ITL p50/p95/p99, goodput at SLO
   └─────────────────┘
                          │
                          ▼  add a second machine
   ┌── 2 g5.xlarge ──┐
   │                 │
   │           ┌──► replica 1 (g5.xlarge)
   │  client ──► vllm-router (prefix-aware)
   │           └──► replica 2 (g5.xlarge)
   │                 │
   │  expect:  1.6× – 1.9× the single-replica RPS at the same SLO
   │           (router overhead + cold-cache fill on replica 2)
   └─────────────────┘
                          │
                          ▼  scale to five
   ┌── 5 g5.xlarge ──┐
   │                 │
   │           ┌──► replica 1
   │           ├──► replica 2
   │  client ──► vllm-router ──► replica 3
   │           ├──► replica 4
   │           └──► replica 5
   │                 │
   │  watch for:                                                  │
   │   - per-replica request count spread (Gini < 0.2)            │
   │   - cache hit rate consistent across replicas                │
   │   - p99 TTFT max-vs-min across replicas                      │
   │   - goodput plateauing — that's your operating ceiling       │
   └─────────────────┘
```

### Why this shape and not multi-node TP?

Because g5 has no EFA. Repeat as many times as needed:

```
   ┌──────────────────────────────────────────────────────────┐
   │  g5 has no EFA. Multi-node TP across g5 is not viable.   │
   │  More g5 boxes = more independent replicas + a router.   │
   │  Need a bigger model? Step up to g6e.xlarge first.       │
   └──────────────────────────────────────────────────────────┘
```

The full size-up decision tree (when to leave g5 entirely) is in
[[hardware/multi-gpu-options]] and [[hardware/aws-gpu-landscape]]; the
single-GPU upgrade path is [[comparisons/g6e-xlarge-deployment-recipe]].

---

# A pocket cheat-sheet (after you've done the reading)

```
   ┌─ One-liners worth memorizing ─────────────────────────────────┐
   │                                                                │
   │ • TP within a node, PP across nodes.                           │
   │ • g5 has no EFA → no multi-node TP → use replicas + router.    │
   │ • Goodput-at-SLO is the only honest scaling number.            │
   │ • Use open-loop load gen (Poisson) when sizing for SLO.        │
   │ • Prefill = compute-bound. Decode = memory-bandwidth-bound.    │
   │ • Disagg helps long prefills + tight TPOT + heterogeneous GPUs.│
   │ • Default engine: vLLM. Prefix-heavy: SGLang. Tight VRAM:      │
   │   LMDeploy (AWQ INT4). NVIDIA peak: TensorRT-LLM.              │
   │ • Default K8s orchestration: vLLM Production Stack (Helm).     │
   │                                                                │
   └────────────────────────────────────────────────────────────────┘
```

# Glossary (skim once, refer back as needed)

| Term | Meaning |
|---|---|
| **VRAM** | GPU memory. The A10G has 24 GB. |
| **TP / PP / DP / EP** | Tensor / Pipeline / Data / Expert parallel. See [[concepts/parallelism-strategies]]. |
| **NVLink / NVSwitch** | NVIDIA's high-bandwidth GPU-to-GPU link inside a server. A10G has none. |
| **EFA** | AWS's low-latency cross-node network. g5 lacks it. |
| **ENA** | Standard AWS Ethernet. What g5 has between boxes. |
| **TTFT** | Time To First Token. |
| **ITL / TPOT** | Inter-Token Latency / Time Per Output Token. |
| **SLO** | Service Level Objective — the latency promise you make. |
| **Goodput** | RPS that meet the SLO. The one number to scale on. |
| **MBU** | Model Bandwidth Utilization (decode-phase efficiency). |
| **KV cache** | Per-token state stored on the GPU; reused across decode steps. |
| **Prefix cache** | Reusing KV cache across requests that share a prefix. |
| **AWQ / GPTQ / FP8 / NVFP4 / MXFP4** | Quantization schemes. See [[infrastructure/quantization]]. |
| **MoE** | Mixture of Experts. Sparse activation; fewer FLOPs per token at the same parameter count. |
| **PD-disagg** | Prefill / Decode disaggregation. See [[concepts/disaggregated-serving]]. |
| **Router** | Load-balancer that's aware of LLM-specific things (prefix cache, queue, KV usage). |
| **g5 / g6e / p4d / p5 / p6** | AWS GPU instance families. See [[hardware/aws-gpu-landscape]]. |

# Where to go next

- The master comparison table for the wiki's research question:
  [[comparisons/tool-calling-models-on-a10g]].
- Models grouped by budget:
  [[comparisons/models-by-budget]].
- Concrete copy-paste deployment for the next-tier-up GPU:
  [[comparisons/g6e-xlarge-deployment-recipe]].

## Related

- [[overview]]
- [[concepts/parallelism-strategies]]
- [[concepts/serving-performance-measurement]]
- [[concepts/disaggregated-serving]]
- [[hardware/aws-efa]]
- [[infrastructure/serving-stack-landscape]]
- [[infrastructure/triton-vs-dynamo]]
- [[infrastructure/sglang]]
- [[infrastructure/lmdeploy]]
- [[infrastructure/tensorrt-llm]]
- [[infrastructure/tgi]]
- [[infrastructure/nvidia-dynamo]]
- [[infrastructure/vllm-production-stack]]
- [[infrastructure/llm-d]]
- [[infrastructure/leaderworkerset]]
- [[infrastructure/aibrix]]
- [[infrastructure/ray-serve-llm]]
- [[comparisons/scaling-1-to-5-machines]]
- [[comparisons/serving-stacks-comparison]]

## Sources

This page is a synthesis / map; all primary sources are cited on the
underlying content pages it links to. No new claims are introduced here.
