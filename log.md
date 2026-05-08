# Log

Append-only chronological record. Format: `## [YYYY-MM-DD] <op> | <subject>` where op ∈ {bootstrap, ingest, query, lint, refactor, note}.

## [2026-05-07] bootstrap | scaffold + seed initial research pages

- Created schema files: [[CLAUDE.md]] (pointer), [[AGENTS.md]] (source of truth).
- Created [[index.md]] catalog and this log.
- Seeded `wiki/` with overview, hardware (A10G + multi-GPU), infrastructure (vLLM, NVIDIA Dynamo, quantization), concepts (tool selection, code generation, benchmarks), 13 model pages, and the master comparison table at [[comparisons/tool-calling-models-on-a10g]].
- All numbers in seeded pages were marked `*(unverified — needs source)*`.

## [2026-05-07] ingest | first research sweep — 56 primary sources, 10 source pages

Ran a parallel multi-agent web research sweep covering hardware/AWS pricing, vLLM, NVIDIA Dynamo, all major benchmarks (BFCL, SWE-bench Verified, LiveCodeBench, Aider, EvalPlus), and HF model cards / tech reports / vendor blogs for every wiki candidate model.

### Raw sources saved (56 files in `raw/`)
Hardware: aws-g5-instance-types, aws-g5-pricing, aws-p4-instance-types, aws-p5-instance-types, nvidia-a10-datasheet, nvidia-a10g-vs-a10, nvidia-dynamo-readme.
vLLM: vllm-tool-calling-docs (×2), vllm-tool-call-parsers (×2), vllm-quantization-docs, vllm-quantization-ampere, vllm-engine-args, vllm-release-notes-v0.20, vllm-structured-outputs.
Benchmarks: bfcl-leaderboard, bfcl-methodology, bfcl-overview, swebench-verified, swebench-methodology, livecodebench (×2), aider-leaderboard, evalplus-leaderboard.
Models: qwen2.5-coder-{7b,14b,32b}-hf-card + tech-report + blog; deepseek-coder-v2-lite-hf-card + paper; codestral-22b-hf-card + announcement + mnpl-license; llama-3.1-8b-hf-card + community-license; llama-3.3-70b-hf-card; mistral-small-{2501,3.1,3.2}-hf-card + 3 + 3.1 announcements; phi-3-medium-128k-hf-card + phi-4-hf-card; granite-{3.0-announcement,3.1,3.3}-cards; hermes-3-8b-hf-card + hermes-4-70b-hf-card; xlam-{7b-r,7b-fc-r,2-8b-fc-r,2-32b-fc-r}-hf-card + cc-by-nc-license; functionary-readme.

### Source pages created (10, in `wiki/sources/`)
- [[sources/aws-ec2-pricing-2026-05]]
- [[sources/nvidia-a10g-specs]]
- [[sources/nvidia-dynamo-readme]]
- [[sources/vllm-tool-calling-docs]]
- [[sources/vllm-quantization-docs]]
- [[sources/bfcl-leaderboard-2026-05]]
- [[sources/code-benchmarks-2026-05]]
- [[sources/qwen2.5-coder-tech-report]]
- [[sources/deepseek-coder-v2-paper]]
- [[sources/codestral-mnpl-license]]
- [[sources/llama-3.x-cards]]
- [[sources/mistral-small-3.x-cards]]
- [[sources/phi-3-and-phi-4-cards]]
- [[sources/granite-3.x-cards]]
- [[sources/hermes-3-and-4-cards]]
- [[sources/xlam-family-cards]]
- [[sources/functionary-readme]]

### Wiki pages updated
All 22 wiki pages were updated with verified numbers and `[Source: [[sources/...]]]` citations. Replaced every `*(unverified — needs source)*` marker. Updated `last_updated` and `source_count` frontmatter on each.

### Material corrections from the seed
- **A10G FP16 dense compute**: ~70 TF (not 125; that's the data-center A10). AWS's "250 TOPS" claim for G5 appears to reference the A10.
- **Mistral-Small canonical version is 3.2 (2506)** with 128K context and improved tool-calling template — supersedes both 3.1 and 2501.
- **Granite 3.3 supersedes 3.1** — HumanEval went from "not on card" to 89.73; reasoning tags added.
- **DeepSeek-Coder-V2-Lite license IS commercial-OK** — initial seed left this uncertain. DeepSeek License explicitly permits commercial use.
- **All Salesforce xLAM checkpoints (v1 + v2, every size) are CC-BY-NC-4.0** — non-commercial. No exceptions in the family.
- **Codestral-22B v0.1's MNPL is firmly non-commercial.** Codestral Mamba 7B (Apache-2.0) is the commercial alternative.
- **Phi-3-medium and Phi-4 14B both lack vLLM tool-call parsers** — agentic disqualified despite strong reasoning numbers.
- **Functionary needs MeetKai's `server_vllm.py` fork** — no mainline vLLM parser as of 0.20.1.
- **NVIDIA Dynamo's own README**: "If you're running a single model on a single GPU, your inference engine alone is probably sufficient." Single-A10G value is essentially nil.
- **vLLM tool-call parsers expanded substantially** since the seed: added `xlam`, `deepseek_v3`/`v31`, `qwen3_xml`, `granite4`, `llama4_pythonic`, `openai`, `kimi_k2`, plus ~10 more model-family-specific parsers. Names purely additive across 2025–2026.
- **Qwen2.5-Coder is absent from BFCL** despite tool-calling claims; vLLM `hermes` parser is known-flaky on the Coder variant (issues #10952, #29192, #32926).

### Pages touched (28 wiki + 17 source = 45)
- All 13 model pages
- 2 hardware pages (a10g-g5xlarge, multi-gpu-options)
- 3 infrastructure pages (vllm, nvidia-dynamo, quantization)
- 3 concept pages (tool-selection, code-generation, benchmarks)
- 1 master comparison
- 1 overview
- 1 index
- 17 source pages (10 standalone + 7 model-family consolidated)

## [2026-05-07] ingest | AWS GPU spectrum + frontier open-source models expansion

Broadened the wiki beyond the original A10G-locked scope. Ingested AWS GPU
families G4dn / G6 / G6e / P6-B200 (G5 / P4 / P5 already covered) and the major
open-source LLM releases since the first sweep: Qwen3 / Qwen3-Coder, DeepSeek
V3.1 + R1, Llama 4 Scout/Maverick, GLM-4.5 / 4.5-Air, Kimi K2 / K2.5 / K2.6,
OpenAI gpt-oss-20b / 120b, Mistral Devstral / Devstral-2.

Sources added (raw → wiki/sources):
- raw/aws-g4dn-instance-types.md, raw/aws-g6-instance-types.md, raw/aws-g6e-instance-types.md, raw/aws-p6-b200-instance.md, raw/nvidia-l4-l40s-datasheet.md
- raw/qwen3-coder-cards.md, raw/qwen3-dense-cards.md
- raw/deepseek-v3-r1-family.md
- raw/llama-4-cards.md
- raw/glm-4.5-cards.md
- raw/kimi-k2-cards.md
- raw/gpt-oss-cards.md
- raw/devstral-small-cards.md
- wiki/sources/aws-extended-gpu-pricing-2026-05.md
- wiki/sources/nvidia-l4-l40s-specs.md
- wiki/sources/qwen3-coder-cards.md, wiki/sources/qwen3-dense-cards.md
- wiki/sources/deepseek-v3-r1-family.md
- wiki/sources/llama-4-cards.md
- wiki/sources/glm-4.5-cards.md
- wiki/sources/kimi-k2-cards.md
- wiki/sources/gpt-oss-cards.md
- wiki/sources/devstral-small-cards.md

Model pages added:
- wiki/models/qwen3-coder-30b-a3b.md (A10G/L40S sweet spot, supersedes Qwen2.5-Coder for new deploys)
- wiki/models/qwen3-coder-480b.md (Apache-2.0 frontier code, p5e/p6 only)
- wiki/models/qwen3-32b.md (Apache-2.0 dense hybrid reasoning)
- wiki/models/devstral-small.md (Apache-2.0 24B, SWE-V 53.6→68; replaces Codestral-22B for commercial use)
- wiki/models/glm-4.5-air.md (MIT 106B/12B MoE; g6e.12xlarge sweet spot)
- wiki/models/llama-4-scout.md (Llama 4 community, 109B/17B MoE)
- wiki/models/deepseek-v3.1.md (MIT 671B/37B; SWE-V 66.0; needs H200/B200)
- wiki/models/kimi-k2.md (mod-MIT 1T/32B; K2.6 SWE-V **80.2** — open-source frontier)
- wiki/models/gpt-oss-20b.md (Apache-2.0 21B/3.6B MXFP4)
- wiki/models/gpt-oss-120b.md (Apache-2.0 117B/5.1B MXFP4)

Hardware/landscape pages:
- wiki/hardware/aws-gpu-landscape.md (NEW master menu — full G4dn/G5/G6/G6e/P4/P4de/P5/P5e/P5en/P6 table with sm levels and quant compatibility)
- wiki/hardware/g6e-l40s.md (NEW deep dive on g6e.xlarge — cheapest 48 GB single-GPU AWS box)
- wiki/hardware/multi-gpu-options.md (rewritten — defers pricing to landscape page; adds model-size → smallest-AWS-box decision table)

Comparison artifacts:
- wiki/comparisons/models-by-budget.md (NEW — 6 budget tiers from $0.53/hr to $113.93/hr with best-tool-calling / best-code / best-SWE-V picks per tier)
- wiki/comparisons/tool-calling-models-on-a10g.md (updated — Qwen3-Coder-30B-A3B and Devstral-Small added as new A10G defaults; multi-GPU section extended; recommendations refreshed)

Index + overview:
- wiki/overview.md (broadened scope; added budget-tier table; refreshed candidate landscape; new pitfalls/caveats)
- index.md (split Models into A10G-class vs Multi-GPU/frontier; tagged NEW entries; added all new sources)

Key insights surfaced this round:
1. **g6e.xlarge ($1.861/hr, L40S 48 GB) is the most under-used SKU on AWS** for LLM inference — unlocks 24–32B at FP16/FP8 single-GPU.
2. **AWS sells single-GPU only at 16/24/48 GB** — A100/H100/H200/B200 are 8× chassis only. No middle ground between L40S 48 GB and an 8× H100 box.
3. **MXFP4 weights (gpt-oss) require sm_90+** — on Ampere/Ada vLLM dequantizes to BF16 (~2× VRAM), often defeating the quantization benefit. Practical native AWS = p5+/p6.
4. **Kimi-K2 does NOT fit p5.48xlarge** at FP8 — needs H200 (p5e/p5en) or B200 (p6-b200). This is the single sharpest budget cliff at the top of the AWS lineup.
5. **Qwen2.5-Coder is superseded by Qwen3-Coder** for new deploys; **Codestral 22B is superseded by Devstral-Small** for commercial use. Both successors are Apache-2.0 with mainline vLLM parsers.
6. **Open-source SWE-bench frontier (May 2026)**: Kimi-K2.6 80.2 > Devstral-Small-2-2512 68.0 (24B!) > Qwen3-Coder-480B 66.5 ≈ DeepSeek-V3.1 66.0 > GLM-4.5 64.2 > gpt-oss-20b high 60.7. Devstral and gpt-oss-20b are remarkable: small footprints with frontier-tier scores.

## [2026-05-07] query | next box + model after g5.xlarge

- Recommended **g6e.xlarge** ($1.861/hr, L40S 48 GB) as the next step from g5.xlarge.
- Primary model: **Qwen3-Coder-30B-A3B FP8** (Apache-2.0, MoE 31B/3.3B, mainline `qwen3_coder` parser, full 256K ctx single-GPU).
- Secondary model: **Devstral-Small-2-24B FP8** (Apache-2.0 dense, SWE-V 53.6 → 68.0; FP16 NOT viable on single L40S — only "just fits" weights with no KV).
- Filed copy-paste deployment spec at [[comparisons/g6e-xlarge-deployment-recipe]] for handoff to a deploy agent.
- Added entry to `index.md` under Comparisons.

## [2026-05-08] ingest | alternative serving stacks + multi-machine scaling + perf measurement

Broadened the wiki beyond the original vLLM + NVIDIA Dynamo focus. Added every
credible open-source LLM serving stack (SGLang, TensorRT-LLM, LMDeploy, TGI,
Ray Serve LLM, vLLM Production Stack, llm-d, AIBrix, LeaderWorkerSet),
multi-machine scaling recipe (1 → 2 → 5 g5.xlarge), AWS EFA bandwidth matrix,
multi-node serving performance measurement methodology (TTFT/ITL/goodput,
GenAI-Perf, MLPerf v5.0).

Sources added (raw → wiki/sources):
- raw/serving-stacks-alternatives-2026-05.md
- raw/vllm-distributed-serving-2026-05.md
- raw/nvidia-dynamo-multinode-2026-05.md
- raw/disaggregated-serving-papers-2026-05.md
- raw/k8s-llm-orchestration-2026-05.md
- raw/aws-efa-multinode-2026-05.md
- raw/sglang-distributed-2026-05.md
- raw/llm-serving-perf-metrics-2026-05.md
- raw/llm-serving-perf-tools-2026-05.md
- wiki/sources/serving-stacks-alternatives-2026-05.md
- wiki/sources/vllm-distributed-serving-2026-05.md
- wiki/sources/nvidia-dynamo-multinode-2026-05.md
- wiki/sources/disaggregated-serving-papers-2026-05.md
- wiki/sources/k8s-llm-orchestration-2026-05.md
- wiki/sources/aws-efa-multinode-2026-05.md
- wiki/sources/sglang-distributed-2026-05.md
- wiki/sources/llm-serving-perf-metrics-2026-05.md
- wiki/sources/llm-serving-perf-tools-2026-05.md

Infrastructure pages added:
- wiki/infrastructure/serving-stack-landscape.md (NEW master menu)
- wiki/infrastructure/sglang.md (NEW)
- wiki/infrastructure/tensorrt-llm.md (NEW)
- wiki/infrastructure/lmdeploy.md (NEW)
- wiki/infrastructure/tgi.md (NEW; flagged as maintenance mode)
- wiki/infrastructure/triton-vs-dynamo.md (NEW; disambiguates rename)
- wiki/infrastructure/ray-serve-llm.md (NEW)
- wiki/infrastructure/vllm-production-stack.md (NEW)
- wiki/infrastructure/llm-d.md (NEW)
- wiki/infrastructure/aibrix.md (NEW)
- wiki/infrastructure/leaderworkerset.md (NEW)

Concept pages added:
- wiki/concepts/parallelism-strategies.md (NEW; TP/PP/DP/EP rubric)
- wiki/concepts/disaggregated-serving.md (NEW; DistServe/Splitwise/Mooncake)
- wiki/concepts/serving-performance-measurement.md (NEW; goodput, GenAI-Perf, MLPerf, traps)

Hardware pages added:
- wiki/hardware/aws-efa.md (NEW)

Comparison artifacts added:
- wiki/comparisons/serving-stacks-comparison.md (NEW)
- wiki/comparisons/scaling-1-to-5-machines.md (NEW; the user's specific question)

Updates to existing pages:
- wiki/overview.md (broadened; added scaling/serving-stack threads; new sources)
- wiki/infrastructure/vllm.md (multi-node section; V1 engine; new related links)
- wiki/infrastructure/nvidia-dynamo.md (rename callout; updated perf table; disagg link)
- wiki/hardware/multi-gpu-options.md (multi-node vs multi-replica section; EFA caveat)
- index.md (full update)

Key insights surfaced this round:
1. **g5 has no EFA**, making cross-node tensor parallelism non-viable on the
   wiki's primary baseline. Multi-machine scaling on g5 = independent replicas
   behind a KV-aware router. EFA is available on g6e and above.
2. **Triton was renamed Dynamo-Triton** (March 2025); "Dynamo" brand now spans
   two products (LLM-focused NVIDIA Dynamo + multi-framework Dynamo-Triton).
3. **TGI is in maintenance mode**; HF officially recommends migration to vLLM /
   SGLang / llama.cpp / MLX.
4. **SGLang RadixAttention** materially outperforms vLLM on prefix-heavy /
   agentic / RAG / multi-turn workloads (paper: "up to 6.4× higher
   throughput"). LMSYS Jan 2026: chunked PP yields 3.31× prefill throughput
   on DeepSeek-V3.1 vs TP8.
5. **LMDeploy** is the AWQ-W4A16 specialist: "1.8× vLLM throughput" on
   InternLM2-20B; supports AWQ + KV-quant + prefix cache simultaneously.
   Tool-calling parser coverage is narrow (InternLM2.5, Llama 3.1 only).
6. **Goodput, not throughput**, is the headline multi-node metric (DistServe
   paper). Always measure under stated SLO; use open-loop load generation.
7. **MLPerf Inference v5.0 SLOs** as defaults: Llama 3.1 405B: 99p TTFT ≤ 6s,
   99p TPOT ≤ 175 ms; Llama 2 70B Interactive: 99p TPOT ≤ 40 ms, 99p TTFT
   ≤ 450 ms.
8. **llm-d KV-cache-aware routing**: precise prefix-aware routing yields
   TTFT P90 = 0.542 s vs 31.083 s for approximate — a 57× win. Cached tokens
   are 10× cheaper.
9. **Real-hardware Dynamo at scale**: GB200 NVL72 + Dynamo+vLLM produces
   12,587 tok/s/GPU vs 4,021 tok/s/GPU on a single B200 (SemiAnalysis
   InferenceMAX) — ~3× per-GPU win from disaggregation.
10. **Production Stack** is the canonical answer for "1 → 2 → 5 g5
    machines" — Helm-deployable, prefix-aware routing, LMCache-integrated.

## [2026-05-08] note | beginner LLM-serving learning guide

Added [[wiki/concepts/beginner-llm-serving-guide]] — a top-to-bottom learning
path for someone new to LLM serving who wants to understand the wiki end-to-
end. Mirrors the user-supplied 5-step study plan (foundations → serving-stack
landscape → disaggregation theory → K8s orchestration → 1→2→5 machine
synthesis), translated into beginner vocabulary with many ASCII diagrams.

Pages touched:
- wiki/concepts/beginner-llm-serving-guide.md (new)
- index.md (added under Concepts)

Pure synthesis page; no new claims or sources introduced.
