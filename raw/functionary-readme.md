# MeetKai Functionary — GitHub README + HF cards

source_urls:
- https://github.com/MeetKai/functionary
- https://huggingface.co/meetkai/functionary-small-v3.2
- https://huggingface.co/meetkai/functionary-small-v3.1
fetched: 2026-05-07

## Current variants (mid-2026)

- `meetkai/functionary-v4r-small-preview` — newest, 128k context, code interpreter, generates `<think>` reasoning before tool calls
- `meetkai/functionary-medium-v3.2` — 128k context (medium tier)
- `meetkai/functionary-small-v3.2` — 8B, Llama 3.1 8B base, 128k context, MIT license
- `meetkai/functionary-small-v3.1` — 8B, Llama 3.1 8B base, MIT license, last updated Oct 16 2024

## License

**MIT** for the MeetKai-released checkpoints (notable — much more permissive than xLAM's CC-BY-NC-4.0). However, models built on Llama 3.1 also inherit obligations from the Llama Community License.

## BFCL scores (sourced from project public statements)

- `functionary-medium-v3.1`: 88.88% (ranked 2nd at time of publication)
- `functionary-small-v3.2`: 82.82%
- `functionary-small-v3.1`: 82.53%

(BFCL version not explicitly stated on cards; from timing this is **BFCL v2**.)

## Tool-call format

Two formats coexist across versions:

**v3.2 / TypeScript-injection style:** Function definitions are converted to TypeScript-like `namespace functions { ... }` declarations and injected as the system prompt.

**v3.1 / inline tag style:**

```
<function=get_current_weather>{"location": "Istanbul, Turkey"}</function>
```

This format is **non-standard** vs. Hermes/xLAM JSON-in-XML and is the reason mainline vLLM has historically had spotty support.

## Serving status — vLLM mainline vs MeetKai fork

- **functionary-small-v3.1** model card explicitly says "Mainline vLLM support — no fork required" with `vllm serve "meetkai/functionary-small-v3.1"` working out-of-box.
- **functionary-small-v3.2** model card *does* still say: *"We encourage users to run our models using our OpenAI-compatible vLLM server [https://github.com/MeetKai/functionary]."* — i.e., they prefer their own server wrapper.
- The MeetKai functionary repo provides `server_vllm.py` and `server_sglang.py` wrappers that handle prompt templating + tool-call parsing themselves.
- Mainline vLLM's tool_parsers list (as of 2026) does **not** include a `functionary` parser. Supported parsers: hermes, mistral, llama3_json, llama4_pythonic, granite, granite4, internlm, jamba, xlam, deepseek_v3, deepseek_v31, openai, kimi_k2, pythonic, functiongemma, qwen3_xml, olmo3, gigachat3.

## Recommended serving (from MeetKai's own README)

```bash
# vLLM via MeetKai wrapper
python3 server_vllm.py --model "meetkai/functionary-v4r-small-preview" \
  --host 0.0.0.0 --port 8000 --max-model-len 8192

# SGLang via MeetKai wrapper
python3 server_sglang.py --model-path "meetkai/functionary-v4r-small-preview" \
  --host 0.0.0.0 --port 8000 --context-length 8192
```

Medium models need 4×A6000 or 2×A100-80G with `--tensor-parallel-size 2`.

## Implication

For a single-A10G deployment with **mainline** vLLM + native tool-call parsing, Functionary is the weakest fit of the three specialists: no first-class parser in vLLM core, custom prompt-template requirements, and the project itself routes users through its own server scripts. Hermes (mature `hermes` parser) and xLAM-2 (mainline `xlam` parser) are easier mainline-vLLM citizens.
