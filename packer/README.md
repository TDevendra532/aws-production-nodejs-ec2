# Packer — Custom AMI for Node.js EC2 ASG

This directory will contain the Packer template for building a custom AMI baked with:

- Node.js 20 LTS
- CloudWatch Unified Agent
- systemd service definition for the application
- Log rotation configuration
- Application user (non-root) for running the Node process

## Why custom AMI

User-data-only bootstrap is acceptable for low-frequency deployments but adds 3-4 minutes of boot time per instance (Node install + dependency pulls). With a baked AMI, instances become healthy in under 90 seconds — important for responsive Auto Scaling and Instance Refresh rollouts.

## Build flow (planned)

1. Packer reads `nodejs-base.pkr.hcl` (HCL2 template)
2. Provisions an `amazon-ebs` builder in `ap-south-1`
3. Runs shell/ansible provisioners to install Node, CloudWatch agent, harden the OS
4. Produces an AMI tagged with `app=nodejs-api`, `version=<git-sha>`, `built-at=<timestamp>`
5. AMI ID is emitted as output for the Terraform ASG module to consume

## Notes

- Base AMI: Amazon Linux 2023
- Build runs in CI on every change to this directory
- IMDSv2-only enforcement is set on the Launch Template side, not baked into the AMI
- Secrets are never baked into the AMI — fetched at runtime via user-data + Secrets Manager
