# NVIDIA A10G vs A10 — differences relevant to AWS G5

source_url: https://www.baseten.co/blog/nvidia-a10-vs-a10g-for-ml-model-inference/
fetched: 2026-05-07

## Summary

The A10G (used in AWS G5) is a distinct SKU from the datacenter A10. They share the Ampere architecture, 24 GB GDDR6, and 600 GB/s memory bandwidth, but the A10G has notably lower tensor-core compute and a slight edge in non-tensor FP32.

## Tensor-core compute (FP16, dense)

| GPU  | FP16 tensor TFLOPS (dense) |
|------|---------------------------:|
| A10  |                       125  |
| A10G |                        70  |

Baseten: "the A10G has remarkably lower tensor core compute for every level of precision, from FP32 to INT4."

## Shared specs

- 24 GB GDDR6 VRAM
- 600 GB/s memory bandwidth
- Ampere architecture, sm_86 compute capability
- No NVLink
- No FP8 tensor cores (Ada/Hopper feature)

## Practical implications for LLM inference

Per Baseten: "most model inference for LLMs and similar is memory-bound, not compute-bound." Since memory specs are identical, real-world inference throughput on A10 vs A10G is "comparable" for typical decoder-only LLM workloads. The lower A10G compute matters more for prefill / large-batch / training workloads.

## Open issues

- Exact A10G TFLOPS for BF16, INT8, INT4 not directly published by NVIDIA; only FP16 (70 TF dense) is widely cited.
- AWS-published A10G datasheet PDF (d1.awsstatic.com/.../NVIDIA_AWS_A10G_DataSheet_FINAL_02_17_2022.pdf) was not extractable via webfetch.
