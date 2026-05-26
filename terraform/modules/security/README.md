# Module: security

IAM roles and policies, KMS Customer Managed Keys, Secrets Manager secrets, SSM Parameter Store entries, and security groups for the application tier.

## What this module provisions

### IAM
- EC2 instance profile + role
  - SSM core managed policy (Session Manager)
  - CloudWatch Agent policy
  - Secrets Manager read scoped to specific secret ARNs (no wildcards)
  - SSM Parameter Store read scoped to `/app/*`
  - S3 read for app artifact bucket

### KMS
- Separate Customer Managed Keys (rotation enabled):
  - `kms-ebs` — EBS volume encryption
  - `kms-s3` — S3 bucket encryption (logs, artifacts)
  - `kms-logs` — CloudWatch Logs encryption
  - `kms-secrets` — Secrets Manager encryption

### Secrets & Config
- Secrets Manager secrets:
  - DB credentials placeholder
  - API keys
- SSM Parameter Store entries:
  - Non-sensitive runtime config
  - CloudWatch agent configuration JSON

### Security Groups
- `alb-sg` — inbound 80/443 from `0.0.0.0/0`
- `app-sg` — inbound app port from `alb-sg` only (SG reference, not CIDR)
- `vpce-sg` — 443 from `app-sg` for interface endpoints

### VPC Endpoints
- Interface endpoints: SSM, SSM Messages, EC2 Messages, Secrets Manager
- Gateway endpoint: S3

## Inputs (planned)

| Name | Type | Default | Description |
|------|------|---------|-------------|
| `vpc_id` | string | – | VPC |
| `private_subnet_ids` | list(string) | – | For interface endpoints |
| `private_route_table_ids` | list(string) | – | For S3 gateway endpoint |
| `app_port` | number | `3000` | Node.js app port |
| `secret_arns` | list(string) | `[]` | Secret ARNs the instance role can read |
| `tags` | map(string) | `{}` | Common tags |

## Outputs (planned)

| Name | Description |
|------|-------------|
| `instance_profile_arn` | IAM instance profile ARN (consumed by `asg`) |
| `alb_sg_id` | ALB security group ID |
| `app_sg_id` | App security group ID |
| `kms_ebs_arn` | EBS KMS key ARN |
| `kms_logs_arn` | CloudWatch Logs KMS key ARN |

## Notes

- Separate CMKs per domain so revocation/rotation of one key does not affect unrelated resources
- IAM secrets policy is scoped to specific ARNs — no `Resource: "*"` for `secretsmanager:GetSecretValue`
- VPC endpoints keep routine SSM/Secrets traffic off NAT GW, reducing both cost and surface area
- IMDSv2 enforcement is set on the Launch Template (in `asg` module), not here
