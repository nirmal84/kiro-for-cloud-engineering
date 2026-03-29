---
name: cfn-explain
description: Explain what a CloudFormation template does in plain English, including architecture, costs, and access patterns
---

Explain the CloudFormation template that is currently open or selected.

Produce an explanation structured as follows:

## What this deploys

One paragraph in plain English. No AWS jargon where possible. Describe it as you would to a developer who uses AWS but hasn't read the CloudFormation spec.

Example: "This deploys a containerised web application on ECS Fargate behind an Application Load Balancer. The app runs in private subnets and only accepts HTTPS traffic from the internet. It stores session data in ElastiCache Redis and user data in an RDS PostgreSQL database."

## Architecture diagram (text)

Draw a simple ASCII/text diagram showing how the resources connect:

```
Internet → ALB (HTTPS) → ECS Service → RDS (PostgreSQL)
                               ↓
                         ElastiCache (Redis)
                               ↓
                    S3 (static assets, read-only)
```

## Resource inventory

A table of every resource this creates:

| Resource | Type | Purpose | Monthly cost estimate |
|---|---|---|---|
| ... | AWS::ECS::Service | ... | ~$XX |

For cost estimates, give rough order-of-magnitude figures based on default parameter values. Note which resources have variable costs.

## Access and networking

- Who/what can access each resource (security group analysis)
- Which resources are internet-facing vs private
- What IAM permissions the deployed workload has

## Parameters you'll need to provide

List every parameter that has no default, grouped by:
- Required (no default)
- Optional (has default, may want to change)

## What to watch after deployment

- Key CloudWatch metrics to monitor
- Expected behaviour in the first few minutes after deployment
- Any manual steps needed after stack creation (e.g., run a migration, update DNS)

## Gotchas

Any non-obvious behaviours, costs, or limits someone using this template should know about.
