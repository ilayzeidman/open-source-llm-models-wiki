# DeepSeek V3.x / R1 family — HF cards + tech reports

source_urls:
  - https://huggingface.co/deepseek-ai/DeepSeek-V3.1
  - https://huggingface.co/deepseek-ai/DeepSeek-R1-0528
  - https://api-docs.deepseek.com/news/news250821
  - arXiv: 2412.19437 (V3 tech report)
fetched: 2026-05-07

## DeepSeek-V3.1
- 671B total / 37B active (MoE)
- License: **MIT**
- Context: 128K
- Hybrid reasoning: thinking / non-thinking via prompt template
- Native FP8 (UE8M0 scale format) for weights and activations
  - `mlp.gate.e_score_correction_bias` must be loaded/computed in FP32
- Hardware: 8× H100 80GB or 8× H200 (FP8 native); on Ampere (A100/A10G) requires
  re-quantization to BF16 (~1.34 TB BF16) — practically multi-node only on Ampere

### Benchmarks (V3.1)
| Benchmark | Non-Think | Thinking |
|---|---:|---:|
| MMLU-Redux | 91.8 | 93.7 |
| MMLU-Pro | 83.7 | 84.8 |
| GPQA-Diamond | 74.9 | 80.1 |
| SWE-bench Verified (Agent) | **66.0** | — |
| LiveCodeBench (2408–2505) | 56.4 | 74.8 |
| Aider-Polyglot | 68.4 | 76.3 |
| Codeforces-Div1 (rating) | — | 2091 |
| Terminal-bench (Terminus 1) | 31.3 | — |

V3.1 outperforms V3-0324 by ~+20 pts on SWE-bench Verified (66.0 vs 45.4).

## DeepSeek-R1-0528
- 671B total / 37B active (same arch as V3)
- License: MIT
- Pure reasoning model — always emits `<think>` blocks
- vLLM: `--reasoning-parser deepseek_r1`
- Tool calling: limited support (R1 specialised on math/code reasoning, not agentic FC)

## vLLM tool-call parser
- For V3.x: `--tool-call-parser deepseek_v3` (mainline since vLLM 0.7.x)
- For R1: `--reasoning-parser deepseek_r1` (no tool-call parser; R1 not function-call tuned)

## AWS deployment
- Single-node fit: only on **p5e.48xlarge / p5en.48xlarge (8× H200 141GB = 1.1 TB)**
  for FP8, or **p5.48xlarge (8× H100 80GB = 640 GB)** with quant.
- Cannot run on any G-family AWS instance even at INT4 (~340 GB).
- Multi-node TP+PP across 2× p4d/p5 is the realistic open-source FP16 path.
