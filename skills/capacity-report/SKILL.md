---
name: capacity-report
description: Generate a monthly capacity and cost report for a service or AWS account, with rightsizing recommendations
---

Generate a capacity and cost report. Use the `cost-analysis` MCP tool and `cloudformation` MCP tool where available.

If a service name or stack is not in context, ask: "Which service or AWS account should I report on? Provide the stack name, service name, or account ID."

## Report scope

Ask if not clear:
- **Service report**: one ECS service, Lambda function, or RDS cluster
- **Account report**: all resources in an AWS account for the month

---

## Data to collect (use MCP tools where available)

1. **Cost breakdown** — last 30 days, grouped by service and tag
2. **Compute utilisation** — ECS CPU/memory %, EC2 CPU %, Lambda invocations/duration
3. **Database utilisation** — RDS CPU, connections, storage used vs provisioned
4. **Auto-scaling activity** — scale-out and scale-in events, min/max/average task count
5. **Data transfer** — outbound internet, cross-AZ, NAT Gateway

---

## Output format

### Cost Summary — [Month]

| Category | This month | Last month | Change |
|---|---|---|---|
| Compute (ECS/EC2) | $XXX | $XXX | +/-X% |
| Database (RDS) | $XXX | $XXX | |
| Data transfer | $XXX | $XXX | |
| Storage (S3/EBS) | $XXX | $XXX | |
| Other | $XXX | $XXX | |
| **Total** | **$XXX** | **$XXX** | **+/-X%** |

### Capacity Utilisation

| Resource | Avg utilisation | Peak utilisation | Recommendation |
|---|---|---|---|
| ECS service (CPU) | 42% | 78% | Appropriate — no action |
| ECS service (Memory) | 85% | 95% | Consider increasing memory |
| RDS (CPU) | 12% | 28% | Potential downsize opportunity |
| RDS (Storage) | 45% | 45% | On track — ~8 months until full |

### Rightsizing Recommendations

For any resource with average utilisation <30% (potential downsize) or >70% sustained (potential upsize):

| Resource | Current size | Recommended size | Monthly saving | Action |
|---|---|---|---|---|
| RDS db.r6g.large | db.r6g.large | db.r6g.medium | ~$85/month | Low risk — schedule maintenance window |
| ECS Memory | 2048 MB | 3072 MB | -$12/month | Increase to prevent OOM events |

### Cost Anomalies

Flag any day-over-day cost increases >20% or unusual spend patterns:
- "Data transfer cost spiked on [date] — coincides with [deployment/event]"
- "NAT Gateway cost increased 3x — investigate traffic pattern change"

### Top 5 Optimisation Actions

Ranked by estimated monthly saving:

1. **[Action]** — saves ~$XXX/month — [effort: Low/Medium/High]
2. ...

### Forecast

Based on current trends: next month estimated cost **$XXX** (+/-X% vs this month).

Key assumptions: [list any seasonal or event-driven factors]
