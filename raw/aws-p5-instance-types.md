# AWS EC2 P5 Instance Types (NVIDIA H100 / H200)

source_url: https://aws.amazon.com/ec2/instance-types/p5/
fetched: 2026-05-07

## p5.48xlarge

- GPUs: 8× NVIDIA H100, 80 GB HBM3 each → 640 GB total
- Instance memory: 2 TiB
- Network: 3,200 Gbps EFA
- Local storage: 8× 3.84 TB NVMe SSD (≈30.7 TB)
- EBS bandwidth: 80 Gbps

## p5e.48xlarge

- GPUs: 8× NVIDIA H200, 141 GB HBM3e each → 1,128 GB total
- Instance memory: 2 TiB
- Network: 3,200 Gbps EFA
- Local storage: 8× 3.84 TB NVMe
- EBS bandwidth: 80 Gbps
- Availability: primarily via EC2 Capacity Blocks for ML

## p5en.48xlarge

- Same GPUs as p5e (8× H200 141 GB)
- Instance memory: 2 TiB
- Network: 3,200 Gbps EFA, EBS bandwidth: 100 Gbps (vs p5e's 80 Gbps)
- The "n" variant has improved networking
- Availability: primarily via EC2 Capacity Blocks for ML

## Notes

- All P5 variants are 8-GPU only (no smaller sizes).
- For models requiring H100/H200, no single-GPU AWS option exists; smallest box is p5.48xlarge.
