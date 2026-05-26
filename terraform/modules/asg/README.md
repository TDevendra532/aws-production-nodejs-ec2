# Module: asg

Launch Template + Auto Scaling Group running the Node.js application on a custom Packer-built AMI, with target tracking, ELB health checks, and Instance Refresh for zero-downtime rolling deployments.

## What this module provisions

- 1 × Launch Template (versioned)
  - Custom AMI ID (Node.js 20 LTS, CloudWatch agent pre-installed)
  - Instance type: `t3.medium` (configurable)
  - gp3 EBS, KMS-encrypted
  - IAM instance profile (SSM core, CloudWatch agent, Secrets read scoped to specific ARNs)
  - IMDSv2 enforced
  - No key pair (SSM Session Manager only)
  - User-data: fetch secrets + latest app artifact, start systemd service
- 1 × ASG across 2 AZs
  - `min=2, desired=2, max=6` (production)
  - Health check type: ELB, grace period 180s
  - Target tracking scaling: CPU 60%
  - Optional scheduled scaling for known traffic patterns
- Instance Refresh enabled (min healthy 50%, checkpoints)

## Inputs (planned)

| Name | Type | Default | Description |
|------|------|---------|-------------|
| `vpc_id` | string | – | VPC |
| `private_subnet_ids` | list(string) | – | Private subnets for instances |
| `target_group_arn` | string | – | Target group from `alb` module |
| `app_sg_id` | string | – | App security group (from `security` module) |
| `instance_profile_arn` | string | – | IAM instance profile |
| `ami_id` | string | – | Custom AMI ID from Packer build |
| `instance_type` | string | `t3.medium` | Instance type |
| `min_size` | number | `2` | Minimum capacity |
| `desired_capacity` | number | `2` | Desired capacity |
| `max_size` | number | `6` | Maximum capacity |
| `target_cpu` | number | `60` | Target tracking CPU threshold |
| `tags` | map(string) | `{}` | Common tags |

## Outputs (planned)

| Name | Description |
|------|-------------|
| `asg_name` | ASG name |
| `launch_template_id` | Launch Template ID |
| `launch_template_latest_version` | Latest LT version |

## Notes

- Minimum of 2 instances enforced even at low load — single-instance ≠ HA
- Health check grace period 180s tuned for Node.js cold starts; shorter values caused premature terminations in past experience
- Instance Refresh handles app deploys: update LT version → trigger refresh → rolling replace
- For traffic-shifting deployments, CodeDeploy blue/green is the alternative; not chosen here due to operational simplicity preference for stateless workloads
