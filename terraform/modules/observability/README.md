# Module: observability

CloudWatch log groups, custom metrics infrastructure, alarms wired to SNS, dashboard, and CloudTrail multi-region trail.

## What this module provisions

### Log Groups (KMS-encrypted)
- `/app/nodejs` — application logs (30d retention)
- `/system/syslog` — system logs (14d retention)
- `/aws/vpc/flow-logs` — VPC Flow Logs (30d retention)
- `/aws/waf/logs` — WAF logs (90d retention)

### Custom Metrics
- EMF (Embedded Metric Format) emitted from the Node.js application
- Standard custom metrics: `RequestCount`, `ErrorCount`, `Latency` (p50, p95, p99) per route

### Alarms (wired to SNS topic)
- `cpu-high` — CPU > 80% sustained 5 min
- `alb-5xx` — ALB 5xx count > 10 / minute
- `latency-p95` — target response time p95 > 1s
- `error-rate` — application error rate > 1%
- `asg-unhealthy` — ASG unhealthy host count > 0
- `nat-port-alloc` — NAT GW port allocation errors

### SNS Topic
- Subscriptions: email + Slack webhook (via Lambda subscriber)

### Dashboard
- Single consolidated dashboard with sections:
  - Traffic (ALB metrics)
  - Compute (ASG, EC2)
  - Application (custom EMF metrics)
  - Errors (5xx, application errors, alarm states)
  - WAF (blocked requests, top rules)

### CloudTrail
- Multi-region trail
- Log file integrity validation enabled
- Destination: S3 bucket with Object Lock for tamper protection

## Inputs (planned)

| Name | Type | Default | Description |
|------|------|---------|-------------|
| `alb_arn_suffix` | string | – | ALB ARN suffix for metric dimensions |
| `target_group_arn_suffix` | string | – | TG ARN suffix for metric dimensions |
| `asg_name` | string | – | ASG name for metric dimensions |
| `alarm_email` | string | – | Email subscriber for SNS |
| `slack_webhook_url` | string | – | Optional Slack webhook |
| `log_retention_days` | map(number) | see code | Per-log-group retention overrides |
| `kms_logs_arn` | string | – | KMS key for log encryption |
| `cloudtrail_bucket_arn` | string | – | S3 bucket for CloudTrail |
| `tags` | map(string) | `{}` | Common tags |

## Outputs (planned)

| Name | Description |
|------|-------------|
| `sns_topic_arn` | SNS topic for alarms (other modules can subscribe) |
| `dashboard_name` | CloudWatch dashboard name |

## Notes

- EMF chosen over StatsD — no additional daemon required, native CloudWatch integration via agent log collection
- Dashboard intentionally kept to one page — multi-tab dashboards are not usable during incident response
- Alarm thresholds are starting defaults; expect tuning after first few weeks of production traffic
- Slack delivery uses a small Lambda subscriber on the SNS topic to format messages
