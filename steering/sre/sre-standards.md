---
includeBy:
  - "**/runbooks/**"
  - "**/monitoring/**"
  - "**/alerts/**"
  - "**/dashboards/**"
  - "**/on-call/**"
---

# SRE Standards

## Service reliability tiers

| Tier | SLO | Alerting | On-call | Examples |
|---|---|---|---|---|
| P0 - Critical | 99.95% | PagerDuty, immediate | 24/7 | Payment processing, auth |
| P1 - High | 99.9% | PagerDuty, 15min | Business hours + on-call | Core APIs, order management |
| P2 - Medium | 99.5% | Slack, 1hr | Business hours | Internal tools, reporting |
| P3 - Low | 99.0% | Slack, next business day | Business hours | Batch jobs, analytics |

## SLIs and SLOs

Define SLIs in CloudFormation/IaC alongside the service they measure. Do not define them in a separate monitoring repo.

```yaml
# Standard SLIs for API services
AvailabilityAlarm:
  Type: AWS::CloudWatch::Alarm
  Properties:
    AlarmName: !Sub "${Service}-${Environment}-availability-slo"
    MetricName: HealthyHostCount
    Namespace: AWS/ApplicationELB
    # Alert at 5-minute burn rate before breaching 99.9% monthly SLO
    Threshold: 1
    ComparisonOperator: LessThanThreshold
    EvaluationPeriods: 3
    Period: 60

LatencyAlarm:
  Type: AWS::CloudWatch::Alarm
  Properties:
    AlarmName: !Sub "${Service}-${Environment}-p99-latency-slo"
    MetricName: TargetResponseTime
    ExtendedStatistic: p99
    Namespace: AWS/ApplicationELB
    Threshold: 2.0   # 2 seconds p99 — adjust per service
```

## Runbook requirements

Every P0 and P1 service MUST have a runbook. Runbooks live in the service repo at `docs/runbooks/`.

Minimum runbook structure:
1. **Service overview** — what it does, its dependencies, its SLO
2. **Common alerts** — what each alert means and first-response steps
3. **Escalation path** — who to call if you can't resolve in 30 mins
4. **Key metrics and dashboards** — links to CloudWatch dashboards
5. **Rollback procedure** — exact commands to roll back a bad deployment
6. **Post-mortem template** — pre-filled structure for incident retrospectives

## Alerting standards

### What should alert

- SLO burn rate (availability and latency)
- Error rate above baseline (use anomaly detection, not fixed thresholds)
- Dependency health (downstream service failures)
- Capacity approaching limits (CPU >70% sustained, disk >80%, queue depth)

### What should NOT alert

- Single transient errors
- Anything that resolves itself without human action
- Information you could find on a dashboard (not actionable)
- Anything that fires more than 3 times a week without being acted on (alert fatigue)

### CloudWatch alarm naming

```
{service}-{environment}-{what-is-wrong}

order-service-prod-high-error-rate
payment-api-prod-p99-latency-degraded
platform-network-prod-nat-gateway-exhausted
```

## Deployment standards for SRE

### Blue/green deployments (ECS)

```yaml
MyECSService:
  Type: AWS::ECS::Service
  Properties:
    DeploymentConfiguration:
      MaximumPercent: 200
      MinimumHealthyPercent: 100
      DeploymentCircuitBreaker:
        Enable: true
        Rollback: true    # Auto-rollback on failing health checks
```

### Canary deployments (Lambda)

```yaml
MyAlias:
  Type: AWS::Lambda::Alias
  Properties:
    RoutingConfig:
      AdditionalVersionWeights:
        - FunctionVersion: !GetAtt MyFunction.Version
          FunctionWeight: 0.1   # 10% canary
```

### Rollback triggers

Deployments auto-roll back if:
- CloudWatch alarm in ALARM state for >2 consecutive periods during deployment
- Health check failure rate >5% during deployment
- Any P0 alarm fires during deployment

## Incident response

When a P0/P1 incident fires:

1. **Acknowledge** in PagerDuty within 5 minutes
2. **Create incident channel** in Slack: `#inc-{service}-{YYYY-MM-DD}`
3. **Announce status** — "Investigating high error rate on order-service-prod" in `#incidents`
4. **Diagnose** using the runbook — do not skip steps
5. **Mitigate** — restore service first, understand root cause second
6. **Resolve** and close PagerDuty
7. **Post-mortem** within 48 hours for P0, 1 week for P1

## Capacity planning

Review capacity metrics monthly:
- ECS service CPU/memory utilization (target: 40-60% average)
- RDS instance CPU, storage growth trajectory
- ALB request rate vs instance count
- NAT gateway data transfer costs

Generate capacity reports using the `/capacity-report` skill.
