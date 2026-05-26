# Module: waf

AWS WAF WebACL with managed rule sets, rate-based protection, and logging to S3 via Kinesis Firehose. WebACL is consumed by the `alb` module for attachment.

## What this module provisions

- 1 × WebACL (regional, for ALB)
- AWS Managed rule groups:
  - `AWSManagedRulesCommonRuleSet` (CRS)
  - `AWSManagedRulesKnownBadInputsRuleSet`
  - `AWSManagedRulesSQLiRuleSet`
- Rate-based rule: 2000 requests / 5 min / source IP → block
- WAF logging → Kinesis Firehose → S3 (sampling enabled)

## Inputs (planned)

| Name | Type | Default | Description |
|------|------|---------|-------------|
| `name` | string | – | WebACL name |
| `rate_limit` | number | `2000` | Requests per 5 min per IP |
| `log_bucket_arn` | string | – | S3 bucket for WAF logs |
| `tags` | map(string) | `{}` | Common tags |

## Outputs (planned)

| Name | Description |
|------|-------------|
| `web_acl_arn` | WebACL ARN (consumed by `alb` module) |

## Notes

- Rate limit set at 2000/5min based on prior experience — lower thresholds caused false positives against legitimate batch clients
- Geo-blocking not enabled by default; add via `allowed_countries` variable if scope is region-specific
- WebACL is attached to the ALB in the `alb` module via the WAF association resource, not here
