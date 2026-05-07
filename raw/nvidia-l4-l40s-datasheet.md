# NVIDIA L4 / L40S Datasheets

source_url (L4):   https://www.nvidia.com/en-us/data-center/l4/
source_url (L40S): https://www.nvidia.com/en-us/data-center/l40s/
fetched: 2026-05-07

Both are Ada Lovelace (sm_89) — the inference/professional generation between
A10G (Ampere sm_86) and H100 (Hopper sm_90).

## L4 (used in AWS g6)
- VRAM: 24 GB GDDR6, ~300 GB/s memory bandwidth
- Form factor: single-slot 72 W passive — designed for high density
- FP16 dense: ~121 TFLOPS, FP8 dense: ~242 TFLOPS (with sparsity 2×)
- INT8: ~485 TOPS dense
- **FP8 native**: yes (E4M3, E5M2)
- No NVLink

## L40S (used in AWS g6e)
- VRAM: 48 GB GDDR6, ~864 GB/s memory bandwidth
- Form factor: 350 W dual-slot active
- FP16 dense: ~362 TFLOPS, FP8 dense: ~733 TFLOPS (sparsity 2×)
- INT8: ~733 TOPS dense
- **FP8 native**: yes
- No NVLink

## Why this matters for LLM serving
1. L4 has same VRAM as A10G (24 GB) but lower memory bandwidth (300 vs 600 GB/s)
   → for decode-bound LLM workloads, L4 is typically *slower* than A10G despite
   newer architecture. Where L4 wins: prefill-heavy workloads, FP8 paths.
2. L40S is the single biggest hardware upgrade option *without* leaving Ada/Ampere class:
   2× the VRAM, ~1.4× the memory bandwidth, and FP8 support — at $1.86/hr (g6e.xlarge).
3. Both L4 and L40S enable vLLM's `--quantization fp8` (model.fp8 weights) which is
   blocked on A10G. AWQ INT4 / GPTQ Marlin still works on all three.
