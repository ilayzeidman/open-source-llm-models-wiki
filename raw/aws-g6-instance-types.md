# AWS EC2 G6 Instance Types (NVIDIA L4)

source_url: https://aws.amazon.com/ec2/instance-types/g6/
fetched: 2026-05-07

## GPU
- All G6 instances use NVIDIA L4 Tensor Core GPUs, 24 GB GDDR6 each, Ada Lovelace sm_89
- L4 has FP8 (E4M3/E5M2) tensor cores — vLLM `--quantization fp8` works natively
  on L4 (unlike A10G, sm_86, which lacks FP8)
- L4 dense FP16 ~121 TFLOPS, ~30 TF effective for inference; ~300 GB/s memory bandwidth
- Same 24 GB capacity as A10G but lower memory bandwidth (300 vs 600 GB/s) — generally
  *worse* for decode-bound LLM workloads despite the newer architecture

## Single-GPU lineup (us-east-1 on-demand)
| Size | GPUs | vCPUs | Mem (GiB) | $/hr |
|---|---:|---:|---:|---:|
| g6.xlarge  | 1× L4 | 4 | 16 | $0.8048 |
| g6.2xlarge | 1× L4 | 8 | 32 | $0.9776 |
| g6.4xlarge | 1× L4 | 16 | 64 | $1.3232 |
| g6.8xlarge | 1× L4 | 32 | 128 | $2.0144 |
| g6.16xlarge| 1× L4 | 64 | 256 | $3.3968 |

## Multi-GPU lineup
| Size | GPUs | Total VRAM | $/hr | Interconnect |
|---|---:|---:|---:|---|
| g6.12xlarge | 4× L4 | 96 GB | $4.6024 | PCIe Gen4 |
| g6.24xlarge | 4× L4 | 96 GB | $6.6752 | PCIe Gen4 |
| g6.48xlarge | 8× L4 | 192 GB | $13.3504 | PCIe Gen4 |

## Notes
- No NVLink on G6 — multi-GPU is PCIe-only.
- G6 is *slightly cheaper* than G5 at xlarge ($0.80 vs $1.006) and supports FP8 weights.
- Net positioning: G6 wins on $ at xlarge for short-context inference; G5 wins on
  decode throughput at long context due to higher memory bandwidth.
