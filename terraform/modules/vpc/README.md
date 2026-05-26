# Module: vpc

Creates the foundational network layer: VPC, subnets across 2 AZs, internet gateway, NAT gateways, route tables, and VPC Flow Logs.

## What this module provisions

- 1 × VPC (`/16` CIDR, DNS hostnames + resolution enabled)
- 2 × Public subnets (`/24` each, one per AZ) for ALB and NAT GWs
- 2 × Private subnets (`/22` each, one per AZ) for EC2 app instances
- 2 × Isolated subnets (`/24` each, one per AZ) reserved for future data tier
- 1 × Internet Gateway attached to the VPC
- 2 × NAT Gateways (one per AZ) for private subnet egress
- Route tables: public (→ IGW), private per-AZ (→ NAT GW in same AZ), isolated (no internet route)
- VPC Flow Logs → CloudWatch Logs (30-day retention, KMS-encrypted)
- Default NACLs locked down to least-privilege

## Inputs (planned)

| Name | Type | Default | Description |
|------|------|---------|-------------|
| `vpc_cidr` | string | `10.0.0.0/16` | VPC CIDR block |
| `azs` | list(string) | – | Availability zones to use |
| `enable_flow_logs` | bool | `true` | Toggle Flow Logs delivery |
| `nat_gateway_count` | number | `2` | Set to 1 in staging for cost |
| `tags` | map(string) | `{}` | Common tags merged into all resources |

## Outputs (planned)

| Name | Description |
|------|-------------|
| `vpc_id` | VPC ID |
| `public_subnet_ids` | List of public subnet IDs |
| `private_subnet_ids` | List of private subnet IDs |
| `isolated_subnet_ids` | List of isolated subnet IDs (reserved) |
| `nat_gateway_ips` | EIPs allocated to NAT GWs |

## Notes

- Staging environment can override `nat_gateway_count = 1` to halve NAT cost
- Isolated subnets are provisioned but unused at this stage to avoid re-planning the VPC when RDS/ElastiCache is added later
- Flow Logs retention deliberately short (30 days) — incident triage horizon, not long-term audit
