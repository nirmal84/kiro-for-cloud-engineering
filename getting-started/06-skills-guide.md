# Step 6: Creating Kiro Skills for Cloud Operations

## What are skills?

Skills are reusable, shareable AI workflows you invoke with a slash command. Instead of typing a long prompt every time, you write it once as a skill and call it as `/cfn-review` or `/runbook`.

Skills live in `.kiro/skills/` and are markdown files with a structured prompt.

## Anatomy of a skill

```markdown
---
name: cfn-review
description: Review a CloudFormation template for security, cost, and compliance issues
---

Review the selected CloudFormation template and check for:

1. **Security issues**
   - Over-permissive IAM (wildcards on Action or Resource)
   - Unencrypted storage (S3, EBS, RDS, DynamoDB)
   - Public-facing resources without explicit justification
   - Missing VPC endpoint usage for AWS service access

2. **Cost risks**
   - Resources with no deletion policy in non-prod that should have one
   - Instance types that could be right-sized
   - Multi-AZ where single-AZ would suffice for non-prod

3. **Compliance**
   - Missing required tags: Environment, Team, CostCentre, ManagedBy
   - Missing permission boundaries on IAM roles
   - Resources in wrong regions

Output a markdown table: | Resource | Issue | Severity | Recommendation |
```

## Invoking a skill

```
# In Kiro chat:
/cfn-review

# With context (select a file first, then invoke):
Select template.yaml > /cfn-review
```

## Starter skills in this repo

| Skill | Command | What it does |
|---|---|---|
| [CFN Review](../skills/cloudformation/cfn-review.md) | `/cfn-review` | Security, cost, compliance review |
| [CFN Explain](../skills/cloudformation/cfn-explain.md) | `/cfn-explain` | Plain-English explanation of a template |
| [Cost Estimate](../skills/cost/cost-estimate.md) | `/cost-estimate` | Rough monthly cost estimate |
| [Security Scan](../skills/security/security-scan.md) | `/security-scan` | OWASP/CIS check on IAM and network config |
| [Runbook Generator](../skills/runbooks/runbook-gen.md) | `/runbook` | Generate an SRE runbook for a service |
| [Incident Summary](../skills/runbooks/incident-summary.md) | `/incident` | Summarise an ongoing incident for comms |
| [Drift Check](../skills/cloudformation/drift-check.md) | `/drift-check` | Identify drift from deployed stack |
| [Change Impact](../skills/cloudformation/change-impact.md) | `/change-impact` | Assess blast radius of a template change |

## Install the starter skills

```bash
cp -r skills/ /path/to/your/infra-repo/.kiro/skills/
```

## Tips for writing effective skills

**Be output-format specific.** Tell the skill exactly what format to respond in. "Output a markdown table with columns: Resource | Issue | Severity | Fix" is better than "list the issues".

**Include your org context.** A good skill references your specific tags, role ARNs, and account patterns — not generic AWS examples.

**Chain with MCP tools.** Skills can instruct Kiro to use MCP tools mid-execution:

```markdown
---
name: stack-health
---
Use the cloudformation MCP tool to describe the stack named {stack_name}.
Then use the cost-explorer MCP tool to get the last 30 days cost for resources tagged with Stack={stack_name}.
Report: deployment date, resource count, monthly cost, any failed resources.
```

**Keep them short and focused.** One skill, one job. Don't write a skill that does review AND generates fixes — that's two skills.
