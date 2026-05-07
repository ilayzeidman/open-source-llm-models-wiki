# AWS EC2 P4 Instance Types (NVIDIA A100)

source_url: https://aws.amazon.com/ec2/instance-types/p4/
fetched: 2026-05-07

## p4d.24xlarge

- vCPUs: 96
- Instance memory: 1,152 GiB
- GPUs: 8× NVIDIA A100, 40 GB HBM2 each → 320 GB total GPU memory
- GPU interconnect: NVSwitch with 600 GB/s bidirectional throughput per GPU
- Network: 400 Gbps ENA + EFA
- Local storage: 8× 1 TB NVMe SSD (8 TB)
- EBS bandwidth: 19 Gbps

## p4de.24xlarge

- Same chassis as p4d.24xlarge
- GPUs: 8× NVIDIA A100, 80 GB HBM2e each → 640 GB total GPU memory
- All other specs identical to p4d.24xlarge

## Key differences

- p4de = 80 GB A100 (HBM2e), p4d = 40 GB A100 (HBM2). 2× GPU memory at higher cost.
- Both expose full 8-way NVSwitch fabric — well suited to tensor-parallel inference of large models.
