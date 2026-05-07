# AWS EC2 P6-B200 Instance (NVIDIA Blackwell)

source_url: https://aws.amazon.com/ec2/instance-types/p6/
source_url: https://aws.amazon.com/blogs/aws/new-amazon-ec2-p6-b200-instances-powered-by-nvidia-blackwell-gpus-to-accelerate-ai-innovations/
fetched: 2026-05-07

## p6-b200.48xlarge
- GPUs: 8× NVIDIA B200 Blackwell, 180 GB HBM3e each → **1,440 GB total**
- vCPUs: 192 (5th-gen Intel Xeon "Emerald Rapids")
- System memory: 2 TiB
- Storage: 8× 3.84 TB NVMe SSD (≈30 TB)
- GPU P2P bandwidth: 1,800 GB/s (NVLink 5)
- Network: 8× 400 Gbps EFA
- EBS bandwidth: 100 Gbps

## Pricing
- On-demand list (us-east-1): **$113.9328/hr**  *(per Vantage / AWS pricing API; subject to AWS update July 2026)*
- Savings Plans: roughly 30% off on-demand
- Spot (us-east-1): from ~$40.31/hr observed
- EC2 Capacity Blocks for ML available; standard reservations 1–14 / 21 / 28 days
  or weekly multiples up to 182 days
- GA since 2025-05-15

## Architecture / precision
- Blackwell adds **FP4 (NVFP4 / MXFP4) tensor cores** in addition to FP8/BF16/FP16 — 2× the
  TFLOPS per tensor core vs Hopper at FP8, plus 4× at FP4.
- vLLM "gpt-oss" path (which uses MXFP4 MoE weights) requires Hopper or newer; B200 supports
  it natively (and faster).

## Region availability
- US West (Oregon) primarily — Capacity Blocks workflow.
- Listed availability also in some other regions for select Savings-Plan / on-demand.

## Comparison: vs P5 H100
- AWS quote: "up to 2× the performance for AI training (time to train) and inference (tokens/sec)"
  vs P5en (H200). 125% more GPU TFLOPS, +27% memory size, +60% memory bandwidth.
