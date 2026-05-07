# AWS EC2 G5 Instance Types (NVIDIA A10G)

source_url: https://aws.amazon.com/ec2/instance-types/g5/
fetched: 2026-05-07

## Full instance specs table

| Instance     | vCPUs | GPUs        | GPU Memory | Instance Memory | Local Storage | Network BW    |
|--------------|------:|-------------|-----------:|----------------:|---------------|---------------|
| g5.xlarge    |     4 | 1× A10G     |      24 GB |          16 GiB | 1× 250 GB NVMe| Up to 10 Gbps |
| g5.2xlarge   |     8 | 1× A10G     |      24 GB |          32 GiB | 1× 450 GB NVMe| Up to 10 Gbps |
| g5.4xlarge   |    16 | 1× A10G     |      24 GB |          64 GiB | 1× 600 GB NVMe| Up to 25 Gbps |
| g5.8xlarge   |    32 | 1× A10G     |      24 GB |         128 GiB | 1× 900 GB NVMe| 25 Gbps       |
| g5.16xlarge  |    64 | 1× A10G     |      24 GB |         256 GiB | 1× 1900 GB NVMe| 25 Gbps      |
| g5.12xlarge  |    48 | 4× A10G     |      96 GB |         192 GiB | 1× 3800 GB NVMe| 40 Gbps      |
| g5.24xlarge  |    96 | 4× A10G     |      96 GB |         384 GiB | 1× 3800 GB NVMe| 50 Gbps      |
| g5.48xlarge  |   192 | 8× A10G     |     192 GB |         768 GiB | 2× 3800 GB NVMe| 100 Gbps     |

## GPU details (per AWS product page)

- A10G Tensor Core GPU
- 80 ray-tracing cores per GPU
- 320 third-generation NVIDIA Tensor Cores per GPU
- "Up to 250 TOPS" (this corresponds to INT8 tensor compute)
- 24 GB GPU memory per A10G

## NVLink

The AWS G5 product page makes no mention of NVLink between A10Gs.
A10/A10G GPUs do not support NVLink (Lenovo A10 product guide explicitly: "NVLink: Not supported"). Multi-GPU G5 boxes therefore communicate over PCIe Gen4 only.

## Notes

- g5.xlarge → g5.16xlarge are all single-GPU; differ only in vCPU/RAM/network.
- g5.12xlarge and g5.24xlarge have 4× A10G; g5.48xlarge has 8× A10G.
- All A10Gs in G5 are PCIe-attached; no NVSwitch.
