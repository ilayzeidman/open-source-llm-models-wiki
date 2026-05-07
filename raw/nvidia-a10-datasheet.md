# NVIDIA A10 Tensor Core GPU — Datasheet

source_url: https://www.nvidia.com/en-us/data-center/products/a10-gpu/
secondary_url: https://lenovopress.lenovo.com/lp1816-thinksystem-nvidia-a10-24gb-pcie-gen4-passive-gpu
fetched: 2026-05-07

This is the datacenter A10 (150 W). The AWS A10G is a related but distinct SKU — see `nvidia-a10g-vs-a10.md`.

## Compute

| Precision        | Without sparsity | With 2:4 sparsity |
|------------------|-----------------:|------------------:|
| FP32 (non-tensor)|        31.2 TFLOPS|                  —|
| TF32 (tensor)    |        62.5 TFLOPS|        125 TFLOPS |
| BF16 (tensor)    |         125 TFLOPS|        250 TFLOPS |
| FP16 (tensor)    |         125 TFLOPS|        250 TFLOPS |
| INT8  (tensor)   |          250 TOPS |          500 TOPS |
| INT4  (tensor)   |          500 TOPS |        1,000 TOPS |

## Memory

- 24 GB GDDR6
- 600 GB/s memory bandwidth
- ECC: supported

## Architecture / cores

- Architecture: NVIDIA Ampere (compute capability sm_86 — not stated on this datasheet, but A10 is GA102, sm_86)
- 9,216 FP32 CUDA cores
- 72 second-generation RT cores
- 320 third-generation Tensor Cores
- 1 NVENC + 2 NVDEC (incl. AV1 decode)

## Physical / interconnect

- Form factor: PCIe full-height, full-length, single slot, passive
- Power: 150 W TDP
- Interface: PCIe Gen4 x16 (64 GB/s)
- NVLink: NOT supported on A10/A10G

No FP8 tensor cores — FP8 was introduced on Hopper (sm_90). On Ampere, FP8 in software stacks is emulated, not native.
