# Step 5: Central Guardrails for AWS Account Access and Deployments

## The problem this solves

Without guardrails, every engineer using Kiro can ask it to generate IAM policies, deployment pipelines, or account-level changes with no awareness of your org's constraints. With central guardrails encoded in steering files, Kiro will:

- Refuse to generate IAM policies that bypass permission boundaries
- Always reference the correct deployment role ARNs for your org
- Generate deployment pipelines that go through your approval gates
- Know which accounts are prod vs non-prod and apply different standards

## Two layers of guardrails

### Layer 1: Kiro steering guardrails (AI-level)
Steering files that tell Kiro what NOT to generate and what patterns to follow. These are advisory — they shape AI output but don't enforce at runtime.

### Layer 2: AWS-level guardrails (infrastructure-level)
SCPs, IAM permission boundaries, CloudTrail, Config rules — the actual enforcement. Kiro steering should reference these and help you generate them correctly.

**Both layers are necessary.** Steering without AWS enforcement is just documentation. AWS enforcement without steering means engineers fight against guardrails they don't understand.

## What to encode in steering guardrails

### Account access patterns

```markdown
# Account Access Guardrails

## Deployment identity
- All deployments use OIDC federation from GitHub Actions — never use long-lived access keys
- Deployment role ARN pattern: arn:aws:iam::{ACCOUNT_ID}:role/GitHubActions-Deploy-{env}
- Never generate policies that allow iam:CreateUser or iam:CreateAccessKey in prod accounts
- Cross-account access uses assume role, not resource policies alone

## Permission boundaries
- All new IAM roles must include: PermissionsBoundary: !Sub arn:aws:iam::${AWS::AccountId}:policy/OrgPermissionBoundary
- Never generate a role without a permission boundary
- Never generate a policy with "Resource": "*" and "Action": "*"
```

### Deployment gates

```markdown
# Deployment Guardrails

## Prod deployments require
1. Change record in ServiceNow (ticket ID in commit message)
2. Two-person approval in GitHub (CODEOWNERS enforced)
3. Deployment only during business hours (08:00-18:00 UTC)
4. Rollback plan documented in PR description

## Generate deployment pipelines with these gates
- Use aws-actions/configure-aws-credentials@v4 with OIDC (never access keys)
- Include cfn-lint and cfn-nag checks before deploy
- Include a manual approval step for prod
- Always deploy to staging first (same template, different params)
```

## Central steering file distribution

The challenge: how do you get org-wide guardrails into every team's repo without everyone manually copying files?

### Option A: Git submodule (recommended)

```bash
# Platform team maintains a central steering repo
git submodule add git@github.com:your-org/kiro-platform-steering.git .kiro/org-steering

# In each team repo's .kiro/settings.json
{
  "steeringPaths": [
    ".kiro/steering",           // repo-specific steering
    ".kiro/org-steering"        // org-wide steering (submodule)
  ]
}
```

Update the submodule across all repos with a GitHub Actions workflow.

### Option B: Cookiecutter/template repo

Include the org steering files in your repo template. Teams get them on creation. They drift over time without automation to keep them in sync.

### Option C: CI enforcement

Even if Kiro steering isn't centralised, you can enforce the same rules in CI with cfn-nag, checkov, or OPA policies. This is the backstop regardless of which option you choose.

## Starter guardrail steering files

| File | What it prevents |
|---|---|
| [guardrails/account-baseline/iam-guardrails.md](../guardrails/account-baseline/) | Over-permissive IAM, missing boundaries |
| [guardrails/deployment/pipeline-standards.md](../guardrails/deployment/) | Hardcoded credentials, missing approval gates |
| [guardrails/account-baseline/network-guardrails.md](../guardrails/account-baseline/) | Public exposure of private resources |

## Next step

[06 - Create your first skill](./06-skills-guide.md)
