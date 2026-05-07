---
tags: [models, tool-calling, specialist, meetkai]
params: 8B
active_params: 8B
license: MIT (MeetKai weights); Llama 3.1 Community on the base
context: 128k
release_date: 2024-10 (collection last updated)
last_updated: 2026-05-07
source_count: 3
---

# Functionary-Small (MeetKai)

MeetKai's tool-calling specialist family. Fine-tuned on Llama 3.1 8B Instruct. **MIT license** on MeetKai weights — most permissive of the three tool-calling-specialist vendors. **The catch**: no mainline vLLM parser — operationally awkward.

- HF: `meetkai/functionary-small-v3.2` ([model card](https://huggingface.co/meetkai/functionary-small-v3.2)) — recommended stable
- Older: `meetkai/functionary-small-v3.1`
- Preview: `meetkai/functionary-v4r-small-preview` (reasoning-first)
- Base: `meta-llama/Meta-Llama-3.1-8B-Instruct`

## Fit on A10G (24 GB)
- FP16: ~16 GB → ✅ ~6 GB KV
- AWQ INT4: ~5 GB → ✅ huge KV budget
- **Verdict**: ✅ comfortable on g5.xlarge

## Strengths

- Tool calling: dedicated focus, OpenAI-compatible output
- Parallel tool calls supported
- **MIT license** on MeetKai weights — commercial-friendly (modulo Llama 3.1 Community License on the base)
- 128k context

### BFCL scores (MeetKai self-report; v2-era)

| Variant | BFCL | Version |
|---|---|---|
| functionary-medium-v3.1 | 88.88 | v2-era |
| functionary-small-v3.2 | 82.82 | v2-era |
| functionary-small-v3.1 | 82.53 | v2-era |

[Source: [[sources/functionary-readme]], [[sources/bfcl-leaderboard-2026-05]]]

## Weaknesses
- ⚠ **No `functionary` parser in vLLM mainline as of 0.20.1.** [Source: [[sources/vllm-tool-calling-docs]]]
  - Card recommends MeetKai's own `server_vllm.py` wrapper from [github.com/MeetKai/functionary](https://github.com/MeetKai/functionary), not bare mainline vLLM.
  - SGLang via `server_sglang.py` is also supported.
  - Alternative: `--tool-parser-plugin` route with a custom plugin.
- This makes Functionary **operationally awkward** for an off-the-shelf vLLM deployment. [[models/hermes-3-llama-3.1-8b]] (also Llama-3.1-8B-based) is a friendlier choice for the same A10G slot.
- Tool-call format differs from Hermes/xLAM:
  - v3.2: TypeScript `namespace functions { ... }` blocks for function definitions
  - v3.1: `<function=name>{...}</function>` inline tag style

## vLLM serving notes
- Path A (recommended by MeetKai): clone the functionary repo, run `python server_vllm.py --model meetkai/functionary-small-v3.2`. This wraps vLLM with MeetKai's prompt templating.
- Path B: `--tool-parser-plugin <custom>.py` with a community plugin matching the v3.x format.
- Path C: bare `vllm serve meetkai/functionary-small-v3.1` works at the model level but **tool-call parsing is not native** — you'll get the raw `<function=...>` text and have to parse it yourself.

[Source: [[sources/functionary-readme]]]

## Cost (AWS, on-demand, us-east-1)
- Smallest box: **g5.xlarge** ($1.006/hr). [Source: [[sources/aws-ec2-pricing-2026-05]]]

## Successor / current state

`functionary-v4r-small-preview` (reasoning-first, 128K, code interpreter) is in preview. The collection was last updated 2024-10. For the canonical stable choice, stay on `functionary-small-v3.2`.

## Recommendation

If you need MIT-license tool-calling specifically: Functionary is the only credible option in the under-8B class. Otherwise prefer **`meetkai/functionary` only when Hermes-3's Llama Community License is a problem**; for most teams, Hermes-3 is operationally simpler thanks to mainline vLLM support.

## Related
- [[models/xlam-7b]], [[models/hermes-3-llama-3.1-8b]] — other tool-calling specialists
- [[concepts/tool-selection]]
- [[infrastructure/vllm]]
- [[wiki/comparisons/tool-calling-models-on-a10g]]

## Sources
- [[sources/functionary-readme]]
- [[sources/vllm-tool-calling-docs]]
- [[sources/bfcl-leaderboard-2026-05]]
