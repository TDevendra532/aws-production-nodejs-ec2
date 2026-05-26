# AWS Production-Ready Node.js on EC2

> Multi-AZ, WAF-protected, fully observable architecture for running a Node.js REST API on AWS EC2 — built against enterprise production standards.

## Overview

This repository contains the design artifacts and planned IaC layout for a production-grade deployment of a Node.js application on AWS. The architecture moves beyond a single-instance setup and applies real-world AWS practices across networking, security, auto scaling, observability, and cost efficiency.

**Status:** Design artifacts complete. Terraform module implementation in progress.

## Deliverables

| # | Deliverable | File |
|---|---|---|
| D1 | Architecture Diagram | [diagrams/D1_architecture.png](diagrams/D1_architecture.png) |
| D2 | Written Justification | [docs/D2_justification.pdf](docs/D2_justification.pdf) |

## Architecture at a Glance

- **Networking:** Custom VPC (`10.0.0.0/16`), 2 AZs, public/private/isolated subnet tiers, 2 NAT Gateways (one per AZ for HA)
- **Compute:** ASG (min=2, desired=2, max=6) across 2 AZs, custom AMI baked with Packer, t3.medium with gp3 EBS
- **Load Balancing:** ALB with HTTPS (ACM), HTTP→HTTPS redirect, WAF WebACL attached
- **Security:** SSM Session Manager (no SSH), Secrets Manager + KMS CMKs, IMDSv2 enforced, VPC endpoints for SSM/Secrets/S3
- **Observability:** CloudWatch Unified Agent, structured log groups, EMF custom metrics, dashboard, multi-region CloudTrail
- **IaC:** Terraform modular layout, S3 + DynamoDB backend, separate tfvars for staging/production
- **Deployments:** Zero-downtime via ASG Instance Refresh

## Repository Layout

```
aws-production-nodejs-ec2/
├── README.md                    ← you are here
├── diagrams/                    ← architecture diagrams (PNG, drawio source)
├── docs/                        ← justification doc and design notes
├── terraform/
│   ├── modules/                 ← reusable building blocks
│   │   ├── vpc/                 ← VPC, subnets, NAT, IGW, route tables, Flow Logs
│   │   ├── alb/                 ← ALB, target group, ACM, listeners, access logs
│   │   ├── asg/                 ← Launch Template, ASG, scaling policies, Instance Refresh
│   │   ├── waf/                 ← WebACL with managed rules + rate limiting
│   │   ├── security/            ← IAM roles, KMS CMKs, Secrets Manager, SSM endpoints
│   │   └── observability/       ← CloudWatch agent config, log groups, alarms, dashboard
│   └── envs/
│       ├── staging/             ← staging tfvars (1 NAT GW, smaller instances)
│       └── production/          ← production tfvars (2 NAT GWs, full sizing)
├── packer/                      ← Packer template for custom AMI
└── scripts/                     ← helper scripts (bootstrap, validate, etc.)
```

## Design Principles

1. **Multi-AZ by default** — no single-AZ dependencies in the critical path
2. **Least privilege** — security groups reference other security groups, not CIDR blocks; IAM scoped to specific ARNs
3. **No SSH** — all instance access via SSM Session Manager, port 22 closed everywhere
4. **Defaults that fail safe** — KMS encryption on EBS/S3/Logs/Secrets, IMDSv2 required, deny-by-default NACLs
5. **Cost-aware** — VPC endpoints to reduce NAT traffic; separate tfvars allow staging to run leaner than production
6. **Operations-first observability** — alarms and dashboards designed for incident response, not vanity metrics

## Key Decisions

Full reasoning is in the [justification doc](docs/D2_justification.pdf). Headline trade-offs:

- **2 NAT Gateways over 1** — additional cost, but removes cross-AZ failure dependency
- **Custom AMI over user-data only** — slower iteration on base image, faster scale-out (~90s vs 3-4 min)
- **ALB over NLB** — Layer 7 features, native WAF integration, path-based routing for future services
- **Instance Refresh over CodeDeploy** — simpler for stateless workloads; CodeDeploy preferred for traffic-shifting scenarios
- **EMF over StatsD** — no additional daemon, native CloudWatch integration

## Future Work

- Provision RDS Multi-AZ in the reserved isolated subnets when the data tier is needed
- SLI-based release gates in the CD pipeline (validate error rate against error budget before prod rollout)
- Spot instance mix in ASG (e.g. 4 On-Demand + 8 Spot) once traffic patterns are understood
- Cross-region disaster recovery with documented RTO/RPO
- Cost anomaly detection and structured FinOps review

## Author

**Devendra Talhande**
Site Reliability Engineer @ Avahi (AWS Premier Partner)
AWS Certified DevOps Engineer – Professional | 4+ years SRE/DevOps experience

- LinkedIn: [linkedin.com/in/devendra-talhande](#)
- Email: devendra.b.talhande@gmail.com

---

*This repository was created as part of an architecture assignment. The design reflects patterns I have implemented in production at AWS Premier Partner environments.*
