---
tags: [models, code, generalist, qwen]
params: 32B
active_params: 32B
license: Apache-2.0
context: 128k
release_date: 2024-11
last_updated: 2026-05-07
source_count: 0
---

# Qwen2.5-Coder-32B-Instruct

Alibaba's flagship open-source code model at release. Reportedly competitive with frontier closed models on code tasks. **Tight fit on a single A10G** even at INT4.

## Fit on A10G (24 GB)
- FP16/BF16 weights: ~64 GB → ❌ won't fit
- INT8: ~32 GB → ❌ won't fit
- AWQ INT4: ~18–19 GB → ⚠ tight; only ~3 GB KV-cache budget → ~7k effective ctx at batch=1
- **Verdict**: ⚠ runs but with limited context. For real use, prefer [[hardware/multi-gpu-options|g5.12xlarge with TP=2 or TP=4]].

## Strengths
- Code: **HumanEval ~92, LiveCodeBench ~40, SWE-bench Verified ~30%** *(unverified — needs source)*
- Strongest open-source code reasoning that exists below 70B class as of late 2024 / 2025 *(unverified)*
- 128k context, Apache-2.0

## Weaknesses
- A10G fit is tight — context-bound at AWQ INT4
- Slower decode at INT4 than 14B sibling per request (more weight bytes to stream)
- Best deployment is multi-GPU; single-A10G is a compromise

## vLLM serving notes
- `--tool-call-parser hermes` *(unverified)*
- Set `--max-model-len ~8192` and `--gpu-memory-utilization 0.95` for single-A10G to maximize KV cache
- Consider TP=2 on g5.12xlarge for FP16-quality + headroom

## Cost (AWS, on-demand, us-east-1)
- Single-A10G (compromise): **g5.xlarge** ~$1.006/hr
- Recommended (full quality, comfortable ctx): **g5.12xlarge** ~$5.67/hr (~$4,140/mo) at TP=4 FP16
- Or TP=2 INT8 on g5.12xlarge

## Related
- [[models/qwen2.5-coder-14b]] — fallback if 32B fit is too tight
- [[hardware/multi-gpu-options]]
- [[infrastructure/quantization]]
- [[comparisons/tool-calling-models-on-a10g]]

## Sources
- (none yet)

## TODO / verify
- HF model card: Qwen/Qwen2.5-Coder-32B-Instruct
- Qwen2.5-Coder tech report
- SWE-bench Verified score (much-cited at release)
- Real measurements of context vs throughput on g5.xlarge AWQ INT4
