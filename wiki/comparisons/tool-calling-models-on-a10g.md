---
tags: [comparison, master-table]
last_updated: 2026-05-07
source_count: 0
---

# Tool-Calling & Code Models on A10G — Master Comparison

The artifact that answers the top-level research question from [[wiki/overview]]: which open-source LLMs are credible for tool selection / complex code on a single A10G (24 GB) via [[infrastructure/vllm]] under [[infrastructure/nvidia-dynamo]]?

> All numbers below come from prior knowledge as of cutoff (Jan 2026) and are marked across the seeded model pages as `*(unverified — needs source)*`. **Verify before quoting.** This table is the canonical place to update as sources get ingested.

## How to read

- **VRAM (rec quant)** = expected weight footprint at the recommended quant for A10G fit
- **Max ctx (24 GB, batch=1)** = practical context length with the rest of VRAM as KV cache; lower for batch>1
- **BFCL** = Berkeley Function-Calling Leaderboard (higher = better tool selection); see [[concepts/tool-selection]], [[concepts/benchmarks]]
- **Code (HumanEval / LiveCodeBench)** = single-function / contest-style; see [[concepts/code-generation]]
- **vLLM parser** = `--tool-call-parser` flag value
- **Verdict** = ✅ great fit / ⚠ tight or compromise / ❌ doesn't fit single A10G

## Single-A10G candidates (sorted roughly by tool-calling strength)

| Model | Params | Rec quant | VRAM | Max ctx (1×A10G) | BFCL | HumanEval / LiveCodeBench | License | vLLM parser | $/hr | Verdict |
|---|---|---|---|---|---|---|---|---|---|---|
| [[models/xlam-7b]] | 7B | AWQ INT4 | ~4 GB | 100k+ | top-tier ≤8B *(unv.)* | ~70 / — | cc-by-nc-4.0 (most) | `pythonic` (verify) | $1.01 | ✅ but **non-commercial** |
| [[models/hermes-3-llama-3.1-8b]] | 8B | AWQ INT4 | ~5 GB | 100k+ | high *(unv.)* | ~73 / — | Llama 3.1 community | `hermes` | $1.01 | ✅ commercial-friendly |
| [[models/functionary-small]] | 8B | AWQ INT4 | ~5 GB | 100k+ | high *(unv.)* | ~70 / — | MIT | custom / fork (verify) | $1.01 | ✅ if vLLM supports your variant |
| [[models/granite-3.1-8b]] | 8B | AWQ INT4 | ~5 GB | 100k+ | solid *(unv.)* | ~70 / — | Apache-2.0 | `granite` | $1.01 | ✅ enterprise default |
| [[models/llama-3.1-8b-instruct]] | 8B | AWQ INT4 | ~5 GB | 100k+ | mid-80s *(unv.)* | ~72 / mid-teens | Llama 3.1 community | `llama3_json` | $1.01 | ✅ generalist baseline |
| [[models/qwen2.5-coder-7b]] | 7B | AWQ INT4 | ~5 GB | 100k+ | usable *(unv.)* | **~88 / mid-20s** | Apache-2.0 | `hermes` (verify) | $1.01 | ✅ best 7B for code |
| [[models/qwen2.5-coder-14b]] | 14B | AWQ INT4 | ~9 GB | ~50k | usable *(unv.)* | **~89 / ~33** | Apache-2.0 | `hermes` (verify) | $1.01 | ✅ **sweet spot for code+tools** |
| [[models/deepseek-coder-v2-lite]] | 16B (MoE, 2.4B active) | AWQ INT4 | ~10 GB | ~40k | weak *(unv.)* | ~81 / mid-20s | DeepSeek License (commercial OK) | none — use constrained | $1.01 | ✅ fast decode (MoE) |
| [[models/phi-3-medium-14b]] | 14B | AWQ INT4 | ~9 GB | ~50k | weak *(unv.)* | ~62 / — | MIT | none — use constrained | $1.01 | ⚠ tool calling weak |
| [[models/codestral-22b]] | 22B | AWQ INT4 | ~13 GB | ~30k | weak *(unv.)* | ~82 / mid-20s | MNPL — **non-commercial** | `mistral` (verify) | $1.01 | ⚠ license + tool-calling caveats |
| [[models/mistral-small-24b]] | 24B | AWQ INT4 | ~14 GB | ~30k | mid-80s *(unv.)* | ~75 / — | Apache-2.0 | `mistral` | $1.01 | ✅ generalist 24B |
| [[models/qwen2.5-coder-32b]] | 32B | AWQ INT4 | ~19 GB | **~7k ⚠** | usable *(unv.)* | **~92 / ~40, SWE-bench ~30%** | Apache-2.0 | `hermes` (verify) | $1.01 | ⚠ tight — see multi-GPU |

## Multi-GPU only

| Model | Params | Best fit | Box | $/hr | Notes |
|---|---|---|---|---|---|
| [[models/qwen2.5-coder-32b]] | 32B | TP=2 INT8 or TP=4 FP16 | g5.12xlarge | $5.67 | full-quality 32B coder |
| [[models/llama-3.3-70b-instruct]] | 70B | TP=8 AWQ INT4 | g5.48xlarge | $16.29 | cheapest 70B fit |
| [[models/llama-3.3-70b-instruct]] | 70B | TP=8 FP16 (NVLink) | p4d.24xlarge | $32.77 | best latency |

See [[hardware/multi-gpu-options]] for the full cost menu.

## Recommended starting points by use case

- **Just tool calling, commercial use, cheapest** → [[models/granite-3.1-8b]] or [[models/hermes-3-llama-3.1-8b]] at AWQ INT4 on g5.xlarge.
- **Code + tool calling, commercial use** → [[models/qwen2.5-coder-14b]] at AWQ INT4 on g5.xlarge. **The default recommendation** for the problem statement in [[wiki/overview]].
- **Best tool calling possible, cost no object, single A10G** → [[models/xlam-7b]] (verify license) or [[models/hermes-3-llama-3.1-8b]].
- **Want maximum code quality** → [[models/qwen2.5-coder-32b]] on **g5.12xlarge** (don't compromise context on g5.xlarge).
- **Frontier-quality generalist** → [[models/llama-3.3-70b-instruct]] on g5.48xlarge or p4d.

## Open questions to resolve via ingest

1. Live BFCL v3 (multi-turn) numbers for every model in this table.
2. Real measured throughput on A10G AWQ INT4 (tok/s, $/1M output tokens).
3. SWE-bench Verified scores for each code model.
4. vLLM parser compatibility — what works with mainline vs forks for xLAM and Functionary today.
5. License confirmations — especially xLAM variants and Codestral revisions.
6. Whether Phi-4 / Qwen3 / newer Granite / DeepSeek-Coder-V3 supersede entries here.

## Related
- [[wiki/overview]]
- [[concepts/tool-selection]], [[concepts/code-generation]], [[concepts/benchmarks]]
- [[hardware/a10g-g5xlarge]], [[hardware/multi-gpu-options]]
- [[infrastructure/vllm]], [[infrastructure/nvidia-dynamo]], [[infrastructure/quantization]]

## Sources
- (none yet — ingest first to validate any specific number)
