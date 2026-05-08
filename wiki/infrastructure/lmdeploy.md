---
tags: [infrastructure, serving, lmdeploy, internlm]
last_updated: 2026-05-08
source_count: 1
---

# LMDeploy

The InternLM team's serving toolkit, Apache-2.0. Distinguishing strength:
**AWQ-W4A16 throughput on small GPUs**, with KV-cache quantization and
automatic prefix caching usable simultaneously — most engines forbid this
combination. Directly relevant to fitting 30B+ models on a single A10G or
L40S. [Source: [[sources/serving-stacks-alternatives-2026-05]]]

## Two engines

| Engine | Language | Stance |
|---|---|---|
| **TurboMind** | C++/CUDA | Flagship; "ultimate optimization of inference performance." |
| **PyTorch** | Pure Python | Lower barrier; for experimentation. |

For production, default to TurboMind.

## Headline features (README)

> "persistent batch (a.k.a. continuous batching), blocked KV cache, dynamic
> split & fuse, tensor parallelism, high-performance CUDA kernels."

## Performance claims

- "delivers up to **1.8× higher request throughput than vLLM**" — InternLM2-20B.
- "4-bit inference performance is **2.4× higher than FP16**" (AWQ-W4A16).
- 2025/09 release: "TurboMind MXFP4 on NVIDIA GPUs starting from V100,
  achieving 1.5× the performance of vLLM on H800 for openai gpt-oss models."

## Quantization

- **AWQ-W4A16** — flagship; the reason to choose LMDeploy.
- GPTQ
- FP8 (with MoE optimizations)
- KV-cache INT8 / INT4 quantization
- llm-compressor 4-bit (added 2026/02)

The combination most engines disallow but LMDeploy supports:
**AWQ-W4A16 weights + INT KV cache + automatic prefix caching simultaneously**.
For a 32B model on a 24 GB A10G this is the difference between fits and
doesn't.

## Recent updates

- 2025/01: full DeepSeek V3 + R1 support
- 2025/04: DeepSeek optimizations via FlashMLA + DeepGemm
- 2025/09: TurboMind MXFP4 on NVIDIA V100+
- 2026/02: Qwen3.5 + llm-compressor 4-bit quantization

## Tool calling (limited)

Function calling supported for **InternLM2.5** and **Llama 3.1 (8B/70B)**.
This is materially less coverage than vLLM/SGLang. If you need tool-calling
parsers for Qwen3-Coder, Devstral, GLM-4.5, gpt-oss, Kimi-K2, DeepSeek-V3.1,
LMDeploy is **not** the right choice.

## Multi-GPU / multi-node

"easy and efficient deployment of multi-model services across multiple
machines and cards" via the request-distribution service. TP works; MoE EP
support added recently.

## A10G g5.xlarge fit (excellent)

LMDeploy was built around fitting big models on small GPUs via aggressive
4-bit quantization. The wiki's primary baseline (g5.xlarge / A10G 24 GB) is
exactly the target environment LMDeploy optimizes for.

For the wiki's 30B-class candidates:
- [[models/qwen3-coder-30b-a3b]] AWQ INT4 — fits with comfortable KV budget.
- [[models/qwen2.5-coder-32b]] AWQ INT4 + INT8 KV — LMDeploy fits this where
  vLLM struggles (KV-cache pressure at 32B + 8K ctx).
- [[models/devstral-small]] AWQ INT4 — fits with 80K+ context.

⚠ **But** LMDeploy lacks tool-calling parsers for Qwen3-Coder, Devstral, and
the other 2026 candidates that this wiki targets. **For tool-calling agentic
workloads**, prefer vLLM or SGLang on a g6e.xlarge ($1.861/hr, L40S 48 GB)
where the tighter quantization isn't necessary.

LMDeploy is the right choice when:
- You want maximum throughput on A10G/L40S with AWQ INT4.
- Your model is one of: InternLM family, Llama 3.1, DeepSeek-V3/R1, gpt-oss.
- Tool calling is not your primary capability requirement.

## When to skip LMDeploy

- Tool calling is a primary capability — vLLM/SGLang have dramatically
  broader parser coverage.
- You need Qwen3-Coder / Devstral / GLM-4.5 tool-call parser.
- You want to standardize on a single engine across both code-gen and tool-
  calling workloads — LMDeploy is strong on the former but weak on the latter.

## Related

- [[infrastructure/vllm]]
- [[infrastructure/sglang]]
- [[infrastructure/serving-stack-landscape]]
- [[infrastructure/quantization]]
- [[hardware/a10g-g5xlarge]]

## Sources

- [[sources/serving-stacks-alternatives-2026-05]]
