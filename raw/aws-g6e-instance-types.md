# AWS EC2 G6e Instance Types (NVIDIA L40S)

source_url: https://aws.amazon.com/ec2/instance-types/g6e/
fetched: 2026-05-07

## GPU
- All G6e instances use NVIDIA L40S Tensor Core GPUs, **48 GB GDDR6** each, Ada Lovelace sm_89
- L40S has FP8 native tensor cores; ~362 TFLOPS dense FP16 (much higher than L4/A10G)
- Memory bandwidth ~864 GB/s (HIGHER than A10G's 600 GB/s)
- **Critical**: 48 GB single-GPU capacity unlocks all 24B–32B models at FP16/BF16
  and 70B at AWQ INT4 on a single GPU, without going multi-GPU.

## Single-GPU lineup (us-east-1 on-demand)
| Size | GPUs | VRAM | vCPUs | Mem | Storage | Net | $/hr |
|---|---:|---:|---:|---:|---|---|---:|
| g6e.xlarge   | 1× L40S | 48 GB | 4 | 32 GiB | 250 GB | up to 20 Gbps | $1.861 |
| g6e.2xlarge  | 1× L40S | 48 GB | 8 | 64 GiB | 450 GB | up to 20 Gbps | $2.24208 |
| g6e.4xlarge  | 1× L40S | 48 GB | 16 | 128 GiB | 600 GB | 20 Gbps | $3.00424 |
| g6e.8xlarge  | 1× L40S | 48 GB | 32 | 256 GiB | 900 GB | 25 Gbps | $4.5288 |
| g6e.16xlarge | 1× L40S | 48 GB | 64 | 512 GiB | 1900 GB | 35 Gbps | $7.5776 |

## Multi-GPU lineup
| Size | GPUs | VRAM | vCPUs | Mem | $/hr | Interconnect |
|---|---:|---:|---:|---:|---:|---|
| g6e.12xlarge | 4× L40S | 192 GB | 48 | 384 GiB | $10.4928 | PCIe Gen4 |
| g6e.24xlarge | 4× L40S | 192 GB | 96 | 768 GiB | $15.0656 | PCIe Gen4 |
| g6e.48xlarge | 8× L40S | 384 GB | 192 | 1,536 GiB | $30.13125 | PCIe Gen4 |

## Notes
- No NVLink on G6e either — multi-GPU still PCIe Gen4.
- L40S is the highest-VRAM single-GPU option in any G family at AWS (48 GB vs 24 GB).
- For LLM inference: **g6e.xlarge at $1.861/hr is the cheapest 48 GB single-GPU box on AWS**.
  This unlocks Qwen2.5-Coder-32B at FP16 single-GPU with usable context, and 70B AWQ INT4.
- AWS marketing claim: "most cost-efficient GPU instances for deploying generative AI models".
