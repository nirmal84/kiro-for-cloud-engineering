# Account Baseline Guardrails

This directory contains CloudFormation templates and Kiro steering files for enforcing a consistent security baseline across all AWS accounts in your organisation.

## Deploy this once per account

The account baseline is deployed via your account vending pipeline, not manually. These templates are maintained by the platform team and should not be modified by individual teams.

## What the baseline deploys

### 1. Security foundations (`security-baseline.yaml`)

- AWS Config enabled with required rules
- CloudTrail enabled (org-level trail in management account)
- GuardDuty enrolled to org master
- Security Hub enrolled with AWS Foundational Security Best Practices and CIS Level 2
- IAM Access Analyzer
- Default VPC deleted

### 2. IAM baseline (`iam-baseline.yaml`)

- `OrgPermissionBoundary` policy — attached to all new roles
- `BreakGlassRole` — emergency access with CloudTrail logging alert
- `AuditReadOnlyRole` — for security team audits
- Password policy enforced
- Root account activity alarm

### 3. Network baseline (`network-baseline.yaml`)

- VPC with private/public/data tier subnets across 3 AZs
- Transit Gateway attachment
- VPC endpoints: S3, ECR API, ECR DKR, Secrets Manager, SSM, SSM Messages, EC2 Messages
- VPC Flow Logs to centralized S3 bucket in Log Archive account
- Route53 resolver rules for internal DNS

### 4. Budget baseline (`budget-baseline.yaml`)

- Monthly budget alert at 80% of account allocation
- Anomaly detection alert for day-over-day cost spike >50%
- SNS → Slack notification

## Steering guardrails

The steering files in this directory tell Kiro to:
- Reference the correct permission boundary ARN in all generated roles
- Use the org-standard VPC and subnet SSM paths instead of asking for IDs
- Not recreate any baseline resources in service stacks

## Files

| File | Purpose |
|---|---|
| [iam-guardrails.md](./iam-guardrails.md) | IAM guardrail steering |
| [network-guardrails.md](./network-guardrails.md) | Network guardrail steering |
| [security-baseline.yaml](./security-baseline.yaml) | CFN template: security baseline |
| [iam-baseline.yaml](./iam-baseline.yaml) | CFN template: IAM baseline |
| [network-baseline.yaml](./network-baseline.yaml) | CFN template: network baseline |
