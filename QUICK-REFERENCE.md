# Kiro Cloud Teams — Quick Reference

## Kiro chat shortcuts (copy-paste these)

### Generate infrastructure

```
Create a CloudFormation template for [what you need].
Use our standard tagging, naming, and security patterns.
```

```
Generate a GitHub Actions deployment pipeline for a CloudFormation stack.
Use OIDC federation, include cfn-lint and checkov validation, and deploy staging before prod with a manual approval gate.
```

```
Create an ECS Fargate service for [app name] that connects to an existing RDS database.
The app runs on port 8080, needs to read from S3, and should scale between 2 and 10 tasks.
```

### Review and validate

```
/cfn-review          → Security, compliance, cost review
/security-scan       → CIS/OWASP security scan
/iam-review          → IAM least-privilege analysis
/change-impact       → Blast radius of current changes
/drift-check         → Stack drift detection
```

### Understand and explain

```
/cfn-explain         → Plain-English architecture explanation
/cost-estimate       → Rough monthly cost estimate
/capacity-report     → Monthly utilisation and rightsizing report
```

### SRE and operations

```
/runbook             → Generate a service runbook
/incident            → Incident status update or post-mortem draft
```

---

## Steering files — what to copy to your repo

```bash
# Minimum set for any CloudFormation repo:
.kiro/steering/
  cfn-structure.md     # Template structure rules
  cfn-tagging.md       # Mandatory tags
  cfn-security.md      # Encryption, IAM, network defaults
  cfn-naming.md        # Naming conventions
  cfn-parameters.md    # SSM parameter patterns
  cfn-outputs.md       # Export conventions
  security-guardrails.md   # Never-generate patterns

# Add for platform engineering repos:
  platform-standards.md    # Account structure, cross-account patterns

# Add for SRE/monitoring repos:
  sre-standards.md         # SLOs, alerting, runbook requirements
```

---

## MCP servers — minimum starter set

Add to `.kiro/settings.json`:

```json
{
  "mcpServers": {
    "aws-docs":     { "command": "uvx", "args": ["awslabs.aws-documentation-mcp-server@latest"] },
    "cloudformation": { "command": "uvx", "args": ["awslabs.aws-cloudformation-mcp-server@latest"],
                        "env": { "AWS_PROFILE": "${env:AWS_PROFILE}", "AWS_REGION": "eu-west-1" } },
    "cost-analysis":  { "command": "uvx", "args": ["awslabs.aws-cost-analysis-mcp-server@latest"],
                        "env": { "AWS_PROFILE": "${env:AWS_PROFILE}" } }
  }
}
```

---

## CloudFormation template checklist

Before submitting any CFN PR:

- [ ] Tags: `Environment`, `Team`, `CostCentre`, `ManagedBy`, `Application` on every resource
- [ ] IAM roles: `PermissionsBoundary` pointing to `OrgPermissionBoundary`
- [ ] Storage: encryption enabled on S3, RDS, EBS, DynamoDB
- [ ] S3: `PublicAccessBlockConfiguration` with all four set to `true`
- [ ] RDS: `PubliclyAccessible: false`
- [ ] Security groups: no `0.0.0.0/0` except port 443 on public ALBs
- [ ] No hardcoded account IDs, regions, or credentials
- [ ] `DeletionPolicy: Retain` on stateful resources in prod
- [ ] Outputs for everything another stack might consume
- [ ] SSM parameters written for shared infrastructure values

---

## Deployment pipeline checklist

Before merging any pipeline change:

- [ ] Uses `aws-actions/configure-aws-credentials@v4` with OIDC (no access keys)
- [ ] Runs `cfn-lint` before deploy
- [ ] Runs `cfn-nag` or `checkov` before deploy
- [ ] Deploys staging before prod
- [ ] Prod has a manual approval gate (GitHub environment protection)
- [ ] Smoke tests run after each deployment
- [ ] No secrets hardcoded in workflow YAML

---

## Key file locations

| What | Where |
|---|---|
| VPC ID (prod) | SSM: `/platform/network/prod/vpc-id` |
| Private subnets (prod) | SSM: `/platform/network/prod/private-subnet-ids` |
| Public subnets (prod) | SSM: `/platform/network/prod/public-subnet-ids` |
| Permission boundary | `arn:aws:iam::{account}:policy/OrgPermissionBoundary` |
| Deployment role | `arn:aws:iam::{account}:role/GitHubActions-Deploy-{env}` |
| Log archive bucket | `log-archive-{account-id}-{region}` |
| ECR registry | `{shared-account}.dkr.ecr.eu-west-1.amazonaws.com` |

---

## Common Kiro prompts for cloud work

```
# Investigate a deployed stack
"Use the cloudformation MCP tool to show me the resources in platform-network-prod"

# Cost investigation
"Use the cost-explorer MCP tool to show me what drove the cost increase in eu-west-1 last week"

# Understand a service before modifying it
"Explain what ecs-fargate-service.yaml deploys and what I need to know before changing the task definition"

# Generate with your org's patterns
"Generate a Lambda function that processes SQS messages and writes to DynamoDB.
 Use our standard tags, permission boundary, and Secrets Manager pattern for credentials."
```
