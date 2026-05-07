# AWS EC2 G5/P4/P5 On-Demand Pricing (us-east-1)

source_url: https://instances.vantage.sh/ (per-instance pages, cross-checked vs aws-pricing.com / economize.cloud)
fetched: 2026-05-07
region: us-east-1 (N. Virginia), Linux on-demand

## G5 (NVIDIA A10G, 24 GB GDDR6 each)

| Instance      | GPUs    | Total VRAM | $/hr (on-demand) | $/GPU-hr | $/month (730h) |
|---------------|---------|-----------:|-----------------:|---------:|---------------:|
| g5.xlarge     | 1× A10G |      24 GB |          $1.006  |  $1.006  |       ~$734    |
| g5.2xlarge    | 1× A10G |      24 GB |          $1.212  |  $1.212  |       ~$885    |
| g5.4xlarge    | 1× A10G |      24 GB |          $1.624  |  $1.624  |     ~$1,186    |
| g5.8xlarge    | 1× A10G |      24 GB |          $2.448  |  $2.448  |     ~$1,787    |
| g5.16xlarge   | 1× A10G |      24 GB |          $4.096  |  $4.096  |     ~$2,990    |
| g5.12xlarge   | 4× A10G |      96 GB |          $5.672  |  $1.418  |     ~$4,141    |
| g5.24xlarge   | 4× A10G |      96 GB |          $8.144  |  $2.036  |     ~$5,945    |
| g5.48xlarge   | 8× A10G |     192 GB |         $16.288  |  $2.036  |    ~$11,890    |

Source: instances.vantage.sh per-instance pages, cross-checked.
Note: `g5.xlarge` is the cheapest single-A10G option at $1.006/hr; the $1.212/hr g5.2xlarge buys 2× vCPUs and 2× RAM at the same GPU.

## P4 (NVIDIA A100)

| Instance        | GPUs                | Total VRAM | $/hr (on-demand) | $/GPU-hr |
|-----------------|---------------------|-----------:|-----------------:|---------:|
| p4d.24xlarge    | 8× A100 40 GB HBM2  |     320 GB |         $32.7726 |  $4.097  |
| p4de.24xlarge   | 8× A100 80 GB HBM2e |     640 GB |         $40.9657 |  $5.121  |

Note: vantage.sh originally listed p4d at $21.958 — this is the Reserved/3-year price. The current public on-demand list price for p4d.24xlarge in us-east-1 is $32.7726/hr (per AWS public list); flag for verification. Multiple secondary sources show $32.77.

> ⚠ Verify: get the canonical p4d on-demand $/hr from the official AWS price-list JSON.
> The $21.958 figure from vantage may reflect a Savings-Plan or 1y RI rate.

## P5 (NVIDIA H100 / H200)

| Instance        | GPUs                | Total VRAM | $/hr (on-demand)        | Notes |
|-----------------|---------------------|-----------:|------------------------:|-------|
| p5.48xlarge     | 8× H100 80 GB HBM3  |     640 GB |                $98.32 listed; ($55.04 SP rate quoted) | See note |
| p5e.48xlarge    | 8× H200 141 GB HBM3e|   1,128 GB | Capacity-Blocks-only ($39.799/hr reservation fee) | Not generally on-demand |
| p5en.48xlarge   | 8× H200 141 GB HBM3e|   1,128 GB | Capacity-Blocks-only ($45.768/hr reservation fee) | Better networking (3,200 Gbps EFA) |

Notes:
- p5.48xlarge on-demand list price is $98.32/hr (per AWS public list, multiple cross-refs). The $55.04 figure on vantage is likely a 1y/3y Savings-Plan or RI rate; $98.32 is the canonical on-demand list.
- p5e and p5en are primarily available via EC2 Capacity Blocks for ML, not standard on-demand. Their "$1.8432" on vantage reflects the OS-add-on (RHEL) line item from the Capacity Blocks page, NOT a usable on-demand rate.

## Capacity Blocks reference (reservation fee per hour, US regions)

source_url: https://aws.amazon.com/ec2/capacityblocks/pricing/

| Instance          | Reservation $/hr (US) | GPUs               |
|-------------------|----------------------:|--------------------|
| p5.48xlarge       |              $34.608  | 8× H100            |
| p5e.48xlarge      |              $39.799  | 8× H200            |
| p5en.48xlarge     |              $45.768  | 8× H200            |
| p4d.24xlarge      |              $11.800  | 8× A100 40GB       |
| p4de.24xlarge     |              $14.7663 | 8× A100 80GB       |

## Open issues

- Need authoritative on-demand prices for p4d/p4de/p5 from the AWS price-list API to settle the vantage discrepancy.
- p5e/p5en on-demand availability beyond Capacity Blocks unclear.
