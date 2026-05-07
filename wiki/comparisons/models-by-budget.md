---
tags: [comparison, budget, master-table]
last_updated: 2026-05-07
source_count: 12
---

# Models by AWS budget tier — best open-source LLM per $/hr bracket

The companion artifact to [[comparisons/tool-calling-models-on-a10g]]. Where that
page locks the GPU at A10G and ranks models, this page locks **budget tiers** and
asks: *"For this $/hr, what is the most credible open-source LLM for tool selection
and complex code, and which box should I rent?"*

All prices are us-east-1 on-demand list, May 2026. Spot is typically 30–70% off.

## How tiers are built

- **Tier = single-instance hourly cost** (not multi-node).
- Within a tier, "best for tool calling" and "best for complex code" can differ —
  both are listed where they diverge.
- "Verdict" reflects commercial usability: **🚫 NC license** is a hard blocker;
  **⚠** flags caveats (parser, MoE comms, MXFP4 emulation).
- All claims cite the canonical model page; SWE-bench numbers are vendor-published
  unless marked otherwise.

## The tiers

### Tier 0 — Budget ($0.50–$0.85/hr, ≤ 16–24 GB single GPU)

| Box | GPU | $/hr | Best tool-calling | Best code | Notes |
|---|---|---:|---|---|---|
| g4dn.xlarge | T4 16GB | $0.526 | [[models/granite-3.1-8b\|Granite-3.3-8B]] AWQ INT4 | [[models/qwen2.5-coder-7b]] AWQ INT4 | T4 has no FP8/BF16; slower kernels |
| g6.xlarge | L4 24GB | $0.8048 | [[models/granite-3.1-8b\|Granite-3.3-8B]] | [[models/devstral-small]] (24B) AWQ INT4 | Cheaper than g5; FP8 native; lower mem bw |

**Recommendation**: For ≤ 8B models, g4dn.xlarge wins on $/hr. For 14B–24B, jump to
g5.xlarge or g6.xlarge depending on whether you need A10G's higher memory bandwidth
(better decode throughput at long ctx) or L4's FP8 (better at short ctx + structured outputs).

### Tier 1 — Wiki baseline ($1.00–$1.30/hr, A10G 24 GB)

| Box | $/hr | Best tool-calling | Best code | Sweet spot |
|---|---:|---|---|---|
| **g5.xlarge** | **$1.006** | [[models/granite-3.1-8b\|Granite-3.3-8B]] | [[models/qwen3-coder-30b-a3b]] AWQ INT4 | original baseline |
| g5.2xlarge | $1.212 | [[models/hermes-3-llama-3.1-8b]] | [[models/devstral-small]] | more vCPU/RAM |

**Tier-1 winner for code+tools**: [[models/qwen3-coder-30b-a3b]] AWQ INT4 — replaces
the older [[models/qwen2.5-coder-14b]] recommendation because it ships the new
**`qwen3_coder` parser** that fixes the flaky-`hermes` issue.

**Tier-1 winner for SWE-bench**: [[models/devstral-small]] (Devstral-Small-2 24B) —
**SWE-bench Verified 53.6 → 68.0%** at AWQ INT4, Apache-2.0.

### Tier 2 — 48 GB single GPU ($1.86–$4.50/hr, L40S 48 GB)

| Box | $/hr | Best tool-calling | Best code | Sweet spot |
|---|---:|---|---|---|
| **g6e.xlarge** | **$1.861** | [[models/qwen3-coder-30b-a3b]] FP8 | [[models/devstral-small]] FP16 | **new "default tier"** |
| g6e.2xlarge | $2.242 | same + headroom for batched serving | | |
| g6e.4xlarge | $3.004 | | | extra vCPU/storage |

**This is the single biggest change to the wiki's recommendation**: with 48 GB
on one card, you no longer need to drop to AWQ INT4 for 24–32B models. Devstral
Small at FP16, Qwen3-Coder-30B-A3B at FP8 with full 256K context, or
Mistral-Small-3.2-24B at FP16 — all single-GPU.

### Tier 3 — Mid multi-GPU PCIe ($5–$15/hr)

| Box | GPUs | $/hr | Best for |
|---|---|---:|---|
| g5.12xlarge | 4× A10G | $5.672 | TP=4 32B FP16 / 70B AWQ INT4 |
| g6.12xlarge | 4× L4 | $4.602 | cheapest TP=4 |
| g6e.12xlarge | 4× L40S | $10.493 | **best mid-tier — 192 GB, FP8 native** |

**Tier-3 winner for code+tools**: [[models/glm-4.5-air]] (106B/12B MoE) at **FP8
TP=4** on g6e.12xlarge — MIT license, parent GLM-4.5 scores SWE-V 64.2 / TAU-Bench
70.1 (Air-specific numbers not published), vLLM `glm45` parser. Note: TP=2 does
not fit (106 GB FP8 weights vs 96 GB for 2× L40S).

**Tier-3 winner for cheapest 70B**: [[models/llama-3.3-70b-instruct]] AWQ INT4 TP=4
on g6e.12xlarge ($10.49/hr).

### Tier 4 — High multi-GPU PCIe / cheap NVLink ($15–$33/hr)

| Box | GPUs | $/hr | Best for |
|---|---|---:|---|
| g5.48xlarge | 8× A10G | $16.288 | TP=8 70B FP16 / 100B+ AWQ |
| g6e.24xlarge | 4× L40S | $15.066 | dense 100B FP8 single-node |
| g6e.48xlarge | 8× L40S | $30.131 | TP=8 100B FP8 / 480B INT4 |
| **p4d.24xlarge** | **8× A100 40GB** | **$32.77** | **NVLink + 320 GB** |

**Tier-4 winner**: [[models/llama-4-scout]] INT4 TP=8 on g6e.48xlarge, or
[[models/devstral-small]]-2-123B AWQ INT4 TP=4 on g6e.12xlarge.

### Tier 5 — Frontier multi-node-class ($33–$115/hr)

| Box | GPUs | $/hr | Best for |
|---|---|---:|---|
| p4de.24xlarge | 8× A100 80GB | $40.97 | NVLink, 640 GB |
| p5.48xlarge | 8× H100 80GB | $98.32 | FP8 + MXFP4 native |
| p5e/p5en.48xlarge | 8× H200 141GB | ~$39.8–$45.8 reserve | **Capacity Blocks**; H200 = 1.13 TB |
| **p6-b200.48xlarge** | **8× B200 180GB** | **$113.93** | **FP4 native, 1.44 TB, fastest** |

**Tier-5 winners** (open-source frontier):

| Use | Model | Best box |
|---|---|---|
| Best SWE-bench Verified | [[models/kimi-k2]] (K2.6: **80.2%**) | p5e/p5en (H200 needed for FP8) |
| Best agent + reasoning | [[models/deepseek-v3.1]] (66.0% SWE-V) | p5e/p5en or p6-b200 |
| Best Apache-2.0 frontier | [[models/qwen3-coder-480b]] (66.5% SWE-V) | p5e/p5en or p6-b200 |
| Best MXFP4 reasoning | [[models/gpt-oss-120b]] | p5.48xlarge slice or p6-b200 |
| Best multimodal frontier | [[models/llama-4-scout]] / Maverick | p4d/p5 (Llama 4 Community license) |

## At-a-glance: best open-source per budget

| Budget | Best for **tool calling** | Best for **complex code** | Best for **agent SWE-bench** |
|---|---|---|---|
| < $1/hr | [[models/granite-3.1-8b\|Granite-3.3-8B]] (g4dn.xlarge / g6.xlarge) | [[models/qwen2.5-coder-7b]] (g4dn.xlarge) | [[models/devstral-small]] AWQ (g6.xlarge) |
| $1–$2/hr | [[models/granite-3.1-8b\|Granite-3.3-8B]] (g5.xlarge) | [[models/qwen3-coder-30b-a3b]] (g5.xlarge AWQ) | [[models/devstral-small]] (g5.xlarge AWQ — SWE-V 53.6) |
| $2–$5/hr | [[models/qwen3-coder-30b-a3b]] FP8 (g6e.xlarge) | [[models/qwen3-32b]] AWQ (g6e.xlarge) | [[models/devstral-small]] FP16 (g6e.xlarge — SWE-V 68 with v2-2512) |
| $5–$15/hr | [[models/glm-4.5-air]] FP8 TP=4 (g6e.12xlarge) | [[models/qwen2.5-coder-32b]] FP16 TP=4 (g5.12xlarge) | [[models/glm-4.5-air]] (parent GLM-4.5 SWE-V 64.2; Air-specific unpublished) |
| $15–$33/hr | [[models/llama-3.3-70b-instruct]] (g5.48xlarge) | [[models/devstral-small]]-2-123B (g6e.12xlarge) | [[models/qwen3-coder-480b]] INT4 (g6e.48xl, slow) |
| $33–$115/hr | [[models/qwen3-coder-480b]] (p5e/p5en H200) | [[models/qwen3-coder-480b]] (p5e/p5en) | [[models/kimi-k2]] K2.6 (p5e/p5en or p6-b200) — **SWE-V 80.2%** |

## Important caveats

1. **g5.xlarge remains the wiki baseline** for the original constraint. Tiers below
   and above are the new "expand the spectrum" view.
2. **MXFP4 weights** ([[models/gpt-oss-20b]] / [[models/gpt-oss-120b]]) need sm_90+
   for hardware tensor cores — **practical AWS = p5+ for native speed**. On A10G/L40S
   they dequantize to BF16 (≈ 2× VRAM, no MXFP4 speedup). See [[infrastructure/quantization]].
3. **AWS doesn't sell single H100/H200/B200 SKUs** — minimum is 8× chassis
   ($98–$114/hr). For occasional H100 use, multi-tenant a single 8× box across multiple
   workloads or use Capacity Blocks for shorter durations.
4. **Llama 4 Community License** is a soft blocker: commercial use requires <700M MAU.
   Apache-2.0 alternatives ([[models/qwen3-coder-30b-a3b]], [[models/devstral-small]],
   [[models/qwen3-coder-480b]]) are preferred when the difference is small.
5. **Kimi-K2** does NOT fit p5.48xlarge (8× H100 = 640 GB) at FP8 (~1 TB needed).
   Use p5e/p5en (H200) or p6-b200 only.

## Related

- [[wiki/overview]]
- [[comparisons/tool-calling-models-on-a10g]]
- [[hardware/aws-gpu-landscape]], [[hardware/g6e-l40s]]
- [[infrastructure/quantization]]

## Sources

- [[sources/aws-extended-gpu-pricing-2026-05]]
- [[sources/nvidia-l4-l40s-specs]]
- [[sources/qwen3-coder-cards]]
- [[sources/qwen3-dense-cards]]
- [[sources/deepseek-v3-r1-family]]
- [[sources/llama-4-cards]]
- [[sources/glm-4.5-cards]]
- [[sources/kimi-k2-cards]]
- [[sources/gpt-oss-cards]]
- [[sources/devstral-small-cards]]
- [[sources/bfcl-leaderboard-2026-05]]
- [[sources/code-benchmarks-2026-05]]
