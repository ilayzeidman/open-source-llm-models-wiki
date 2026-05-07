---
tags: [source, models, tool-calling, meetkai]
source_path: raw/functionary-readme.md
source_url: https://github.com/MeetKai/functionary
ingested: 2026-05-07
last_updated: 2026-05-07
---

# MeetKai Functionary — README & current variants

## Key claims

### Variants

| HF model ID | Recommended? |
|---|---|
| `meetkai/functionary-small-v3.2` | **Yes — current stable small** |
| `meetkai/functionary-medium-v3.2` | medium variant |
| `meetkai/functionary-v4r-small-preview` | newest, reasoning-first preview |
| `meetkai/functionary-small-v3.1` | older 3.1 |

- **Small variants** are 8B (base = `meta-llama/Meta-Llama-3.1-8B-Instruct`)
- Context: 128K
- License: **MIT** for MeetKai weights — most permissive of the three tool-calling specialist vendors. Underlying Llama base subject to Llama 3.1 Community License.
- Released: collection last updated 2024-10-16

### Benchmarks (BFCL, MeetKai self-report)

| Variant | BFCL Score | Version |
|---|---|---|
| functionary-medium-v3.1 | 88.88 | v2-era |
| functionary-small-v3.2 | 82.82 | v2-era |
| functionary-small-v3.1 | 82.53 | v2-era |

### Tool-call format
- v3.2: TypeScript `namespace functions { ... }` blocks for function definitions
- v3.1: `<function=name>{...}</function>` inline tag style
- Both differ from Hermes/xLAM JSON formats.

### vLLM mainline support — ⚠ NOT supported

- **No `functionary` parser** in vLLM 0.20.1's tool-parser list.
- `functionary-small-v3.1` card claims `vllm serve` works mainline, but tool-call parsing is *not* native — MeetKai's prompt template handles it.
- **functionary-small-v3.2 card recommends MeetKai's own `server_vllm.py` wrapper** from https://github.com/MeetKai/functionary, not bare mainline vLLM.
- SGLang via `server_sglang.py` is also supported.
- Alternative: `--tool-parser-plugin` route with a custom plugin.

This makes Functionary **operationally awkward** for an off-the-shelf vLLM deployment. Hermes-3 (also Llama-3.1-8B-based, MIT-license-ish via Llama Community) is a friendlier choice for the same A10G slot.

### Successor

`functionary-v4r-small-preview` (reasoning-first, 128K, code interpreter) is in preview.

## Citation URLs
- [HF: functionary-small-v3.2](https://huggingface.co/meetkai/functionary-small-v3.2)
- [HF: functionary-small-v3.1](https://huggingface.co/meetkai/functionary-small-v3.1)
- [GitHub: MeetKai/functionary](https://github.com/MeetKai/functionary)

## Pages updated on ingest
- [[models/functionary-small]]
- [[wiki/comparisons/tool-calling-models-on-a10g]]
