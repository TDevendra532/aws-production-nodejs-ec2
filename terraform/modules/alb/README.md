# Module: alb

Application Load Balancer in public subnets with HTTPS termination, ACM certificate, HTTP→HTTPS redirect, WAF WebACL attachment, and access logs to S3.

## What this module provisions

- 1 × ALB (internet-facing) across both AZs
- Listener on `:443` (HTTPS) → forward to target group
- Listener on `:80` (HTTP) → redirect to HTTPS
- ACM certificate (DNS-validated via Route 53)
- Target group with health check on `GET /health`
- Sticky sessions disabled (app is stateless)
- WAF WebACL association
- Access logs to S3 with KMS encryption + lifecycle policy

## Inputs (planned)

| Name | Type | Default | Description |
|------|------|---------|-------------|
| `vpc_id` | string | – | VPC to deploy into |
| `public_subnet_ids` | list(string) | – | Public subnets for ALB |
| `domain_name` | string | – | DNS name for ACM cert (e.g. `api.example.com`) |
| `route53_zone_id` | string | – | Hosted zone for DNS validation + alias record |
| `log_bucket_arn` | string | – | S3 bucket for access logs |
| `waf_acl_arn` | string | – | WAF WebACL to attach |
| `tags` | map(string) | `{}` | Common tags |

## Outputs (planned)

| Name | Description |
|------|-------------|
| `alb_arn` | ALB ARN |
| `target_group_arn` | Target group ARN (consumed by ASG module) |
| `alb_security_group_id` | SG to reference from app SG |
| `alb_dns_name` | DNS name (alias target) |

## Notes

- Access logs S3 lifecycle: 30d Standard → 90d IA → 1y Glacier → 7y delete
- Health check path is `/health` — application must expose this endpoint returning 200 when ready
- WAF rules live in the `waf` module; this module only attaches the WebACL
