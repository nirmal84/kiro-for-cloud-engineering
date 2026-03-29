---
name: cfn-review
description: Review a CloudFormation template for security issues, cost risks, and compliance violations
---

Review the CloudFormation template that is currently open or selected.

Analyse it across these dimensions and produce a structured report:

## 1. Security Review

Check for:
- IAM policies with `"Action": "*"` or `"Resource": "*"` (wildcard actions/resources)
- IAM roles missing a `PermissionsBoundary`
- S3 buckets missing `PublicAccessBlockConfiguration` with all four settings set to `true`
- S3 buckets missing `BucketEncryption`
- RDS instances with `PubliclyAccessible: true`
- RDS instances with `StorageEncrypted: false` or missing
- EBS volumes with `Encrypted: false` or missing
- Security groups with ingress `CidrIp: "0.0.0.0/0"` on any port other than 443
- Security groups with SSH (22) or RDP (3389) open to any CIDR
- Hardcoded credentials, passwords, or tokens in any property value
- Use of `{{resolve:ssm:` instead of `{{resolve:ssm-secure:` for sensitive values
- Missing `NoEcho: true` on password parameters

## 2. Compliance Review

Check for:
- Missing required tags: `Environment`, `Team`, `CostCentre`, `ManagedBy`, `Application`
- Tags using hardcoded values instead of `!Ref` parameters
- Resource names that don't include the `Environment` parameter
- Missing `DeletionPolicy` and `UpdateReplacePolicy` on stateful resources (RDS, S3, DynamoDB, EFS)
- CloudTrail or Config resources being disabled or deleted

## 3. Cost Risk Review

Check for:
- Multi-AZ or high-spec instances with no condition check for non-prod environments
- Resources that will incur costs even when idle (NAT Gateways, Elastic IPs, RDS stopped instances)
- No auto-scaling configured on ECS services or ASGs
- Over-provisioned instance sizes (e.g., m5.4xlarge as a default)
- S3 buckets with no lifecycle rules for log or backup buckets

## 4. Reliability Review

Check for:
- ECS services with `DeploymentCircuitBreaker` not enabled
- RDS without `MultiAZ: !If [IsProd, true, false]`
- Load balancers with only one availability zone
- Lambda functions with no dead-letter queue configured
- SQS queues with no redrive policy

## Output format

Produce a markdown table:

| Resource Logical ID | Issue | Severity | Recommendation |
|---|---|---|---|
| ... | ... | HIGH/MEDIUM/LOW | ... |

Severity guide:
- **HIGH**: Security vulnerability, hardcoded secret, or publicly accessible sensitive resource
- **MEDIUM**: Compliance violation, missing required tag, no encryption
- **LOW**: Cost risk, reliability gap, missing best practice

After the table, provide a **Summary** with:
- Total issues by severity
- The single most important fix to make first
- Whether this template is safe to deploy to prod as-is (YES/NO with reason)
