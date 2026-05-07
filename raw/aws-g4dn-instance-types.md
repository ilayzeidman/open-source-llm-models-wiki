# AWS EC2 G4dn Instance Types (NVIDIA T4)

source_url: https://aws.amazon.com/ec2/instance-types/g4/
fetched: 2026-05-07

## GPU
- All G4dn variants use NVIDIA T4 Tensor Core GPUs (16 GB GDDR6 each, Turing sm_75)
- T4 is two architectural generations behind A10G; lacks BF16 tensor support
  (only FP16 / INT8 / INT4) and has roughly 65 TF FP16 dense — usable but
  noticeably slower than A10G under vLLM's preferred kernels.

## Single-GPU lineup
| Size | vCPUs | Mem (GiB) | Net | On-demand $/hr (us-east-1) |
|---|---:|---:|---|---:|
| g4dn.xlarge | 4 | 16 | up to 25 Gbps | $0.526 |
| g4dn.2xlarge | 8 | 32 | up to 25 Gbps | $0.752 |
| g4dn.4xlarge | 16 | 64 | up to 25 Gbps | $1.204 |
| g4dn.8xlarge | 32 | 128 | 50 Gbps | $2.176 |
| g4dn.16xlarge | 64 | 256 | 50 Gbps | $4.352 |

## Multi-GPU
| Size | GPUs | Total VRAM | $/hr |
|---|---:|---:|---:|
| g4dn.12xlarge | 4× T4 | 64 GB | $3.912 |
| g4dn.metal | 8× T4 | 128 GB | $7.824 |

## Notes
- Multi-GPU is over PCIe Gen3 — no NVLink.
- vLLM Marlin AWQ kernels run on T4 but at lower throughput than A10G/L4/L40S.
- Cheapest GPU box at AWS for casual inference. Effective ceiling is ~7–8B
  models at AWQ INT4 with short context.
