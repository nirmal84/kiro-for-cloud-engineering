---
name: security-scan
description: Run a comprehensive security scan on CloudFormation templates checking CIS AWS Foundations, NIST controls, and OWASP Cloud Top 10
---

Perform a security scan of the CloudFormation template or IAM policy currently open.

Map findings to the following frameworks and output a prioritised remediation plan.

## Scanning checklist

### Identity and Access Management
- [ ] All IAM roles have permission boundaries attached
- [ ] No `"Action": "*"` with broad resource scope
- [ ] No `"Principal": "*"` in resource-based policies (except public S3 static sites)
- [ ] No IAM users with programmatic access (use roles/OIDC)
- [ ] MFA enforced on all console users (check account-level policy)
- [ ] Cross-account trust uses `ExternalId` condition for third-party roles
- [ ] Service control policies referenced where applicable

### Data Protection
- [ ] All S3 buckets: encryption + public access block + versioning (prod)
- [ ] All RDS: `StorageEncrypted: true` + `PubliclyAccessible: false`
- [ ] All EBS: `Encrypted: true`
- [ ] All DynamoDB: `SSESpecification.SSEEnabled: true`
- [ ] All SQS: `SqsManagedSseEnabled: true` or KMS key
- [ ] All SNS: `KmsMasterKeyId` specified
- [ ] Secrets in Secrets Manager, not SSM String or hardcoded
- [ ] KMS key rotation enabled on customer-managed keys

### Network Security
- [ ] No security group with `0.0.0.0/0` ingress except port 443 on public ALBs
- [ ] No SSH (22) or RDP (3389) open to internet
- [ ] All RDS, ElastiCache in private subnets with no public subnet placement
- [ ] VPC endpoints configured for S3, ECR, Secrets Manager, SSM
- [ ] VPC Flow Logs enabled
- [ ] ALBs have HTTPS listeners and HTTP redirect to HTTPS

### Logging and Monitoring
- [ ] CloudTrail enabled (org-level trail preferred)
- [ ] S3 bucket access logging enabled for sensitive buckets
- [ ] RDS audit logging enabled (prod)
- [ ] ALB access logs enabled
- [ ] CloudWatch alarms for root account usage, failed console logins
- [ ] Config rules enabled in account

### Compute Security
- [ ] EC2/ECS: IMDSv2 required (`HttpTokens: required`)
- [ ] ECS tasks: no `privileged: true` unless absolutely required
- [ ] Lambda: no environment variables with secrets (use Secrets Manager)
- [ ] EC2: SSM agent installed (enables Session Manager — no SSH needed)
- [ ] Container images from trusted ECR registry, not Docker Hub
- [ ] No root user running containers or Lambda functions

## Output format

### Findings

| Control | Status | Severity | Resource | Finding | Fix |
|---|---|---|---|---|---|
| IAM-001 | FAIL | HIGH | MyRole | No permission boundary | Add PermissionsBoundary property |
| NET-003 | FAIL | HIGH | AppSG | Port 22 open to 0.0.0.0/0 | Restrict to VPN CIDR or use SSM |
| ENC-002 | FAIL | MEDIUM | LogsBucket | Encryption not configured | Add BucketEncryption property |
| MON-001 | PASS | | | ALB access logs enabled | |

### Summary

- **CRITICAL (deploy-blocker):** X findings
- **HIGH:** X findings
- **MEDIUM:** X findings
- **LOW/INFORMATIONAL:** X findings

**Compliance posture:**
- CIS AWS Foundations Benchmark: [PASS/FAIL — list failed controls]
- OWASP Cloud Top 10: [Relevant findings mapped to OWASP controls]

### Top 5 fixes (prioritised)

1. [Most critical fix with exact code change]
2. ...

### Auto-fixable items

For findings where the fix is a simple template change, generate the corrected YAML snippet.
