---
tags: [models, generalist, tool-calling, meta, multi-gpu]
params: 70B
active_params: 70B
license: Llama 3.3 Community License
context: 128k
release_date: 2024-12
last_updated: 2026-05-07
source_count: 0
---

# Llama-3.3-70B-Instruct

Meta's 70B model that reportedly closes most of the gap to Llama 3.1 405B at much lower cost. Strong tool calling and reasoning. **Does not fit on a single A10G** in any quantization with usable context.

## Fit on A10G (24 GB)
- FP16: ~140 GB → ❌
- INT8: ~70 GB → ❌
- AWQ INT4: ~40 GB → ❌
- **Verdict**: ❌ multi-GPU only. See [[hardware/multi-gpu-options]].

## Multi-GPU fit
- **g5.48xlarge** (8× A10G = 192 GB): ✅ AWQ INT4 with TP=8; ✅ INT8 with TP=8; FP16 tight
- **p4d.24xlarge** (8× A100 40 GB = 320 GB): ✅ FP16 with TP=8 (NVLink — best latency)
- **p5.48xlarge** (8× H100 80 GB = 640 GB): ✅ FP16 with TP=8; ✅ FP8 native

## Strengths
- Tool calling: among the strongest generalist models *(unverified)*
- Reasoning, math, multilingual — strongest open-source generalist near release
- 128k context, native tool support

## Weaknesses
- Cost: g5.48xlarge ~$16.29/hr ~ $11,900/mo on-demand
- Latency on g5 (PCIe TP=8) is meaningfully worse than p4d (NVLink)

## vLLM serving notes
- `--tool-call-parser llama3_json --tensor-parallel-size 8`
- Use AWQ INT4 to fit on g5.48xlarge with comfortable KV budget

## Cost (AWS, on-demand, us-east-1)
- **g5.48xlarge**: ~$16.29/hr, ~$11,900/mo (cheapest fit)
- **p4d.24xlarge**: ~$32.77/hr — better latency, NVLink
- **p5.48xlarge**: ~$98+/hr — best latency, FP8 native

## Related
- [[models/llama-3.1-8b-instruct]] — small sibling for single-A10G
- [[hardware/multi-gpu-options]]
- [[infrastructure/quantization]]

## Sources
- (none yet)

## TODO / verify
- HF model card: meta-llama/Llama-3.3-70B-Instruct
- Latency comparison: g5.48xlarge vs p4d.24xlarge
- BFCL score
- Real $/1M-token throughput on each box
