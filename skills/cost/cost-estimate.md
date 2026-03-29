---
name: cost-estimate
description: Generate a rough monthly cost estimate for a CloudFormation template or proposed architecture
---

Estimate the monthly AWS cost for the resources in the currently open CloudFormation template or the architecture I describe.

## Instructions

1. Identify every resource that has an ongoing cost (ignore free-tier resources in a real production account)
2. Use the default parameter values where provided, or ask for expected values for critical sizing parameters
3. Assume `eu-west-1` pricing unless the template specifies a different region
4. Provide estimates for two scenarios: **minimum expected** (low traffic, minimum instances) and **typical production** (expected normal load)

## Output format

### Cost breakdown

| Service | Resource | Configuration | Monthly cost (min) | Monthly cost (typical) |
|---|---|---|---|---|
| Amazon ECS | Fargate tasks | 0.5 vCPU, 1GB RAM × 2 tasks | $18 | $36 |
| Amazon RDS | PostgreSQL db.t3.medium | Multi-AZ, 100GB storage | $120 | $120 |
| Application Load Balancer | 1 ALB | ~10M requests/month | $20 | $35 |
| Amazon S3 | 3 buckets | Logging, artifacts, backups | $5 | $20 |
| Data transfer | Outbound | ~50GB/month | $5 | $15 |
| **Total** | | | **$168** | **$226** |

### Cost drivers

List the top 3 cost items and whether they're fixed or variable:
1. **RDS** — $120/month fixed (largest item — consider Aurora Serverless for non-prod)
2. **ECS Fargate** — variable, scales with traffic
3. **Data transfer** — variable, watch NAT Gateway charges

### Cost optimisation opportunities

Based on the template, identify quick wins:
- Is Multi-AZ needed in non-prod? (saves ~50% on RDS)
- Are EC2 instances on On-Demand that could be Savings Plans?
- Any resources that could use Graviton instances (20-40% cheaper)?
- Are S3 lifecycle rules configured for older data?
- Is there a NAT Gateway per AZ when one might suffice for non-prod?

### Cost anomaly risks

Flag anything in the template that could cause unexpected cost spikes:
- Auto-scaling with no maximum capacity set
- S3 buckets with no lifecycle rules that store large amounts of data
- DynamoDB with on-demand billing and no estimated read/write unit projections

### Compare to actual spend (if cost-explorer MCP is available)

Use the cost-explorer MCP tool to fetch the last month's actual spend for resources tagged with matching tags and compare estimate to actuals.
