---
name: iam-review
description: Review IAM roles, policies, and trust relationships in a CloudFormation template for least-privilege compliance
---

Review all IAM resources in the currently open CloudFormation template or IAM policy document.

## IAM resource inventory

First, list every IAM resource in the template:

| Logical ID | Type | Purpose |
|---|---|---|
| ... | AWS::IAM::Role | ... |
| ... | AWS::IAM::Policy | ... |
| ... | AWS::IAM::ManagedPolicy | ... |

## Per-role analysis

For each `AWS::IAM::Role`, evaluate:

### Trust policy
- Who can assume this role? (Principal)
- Are the conditions tight enough? (e.g., OIDC sub conditions, ExternalId for third-party)
- Could an unexpected principal assume this role?

### Permission policy
- What actions are allowed?
- Are resources scoped (not `"*"`)?
- Are there any condition keys that limit when the permission applies?
- Is this the minimum permission needed for the stated purpose?

### Permission boundary
- Is `PermissionsBoundary` present? (Required by org policy)
- Does it reference `OrgPermissionBoundary`?

## Findings format

### Role: [LogicalResourceId]

**Purpose:** [what this role is for, inferred from context]

**Trust analysis:**
- Principal: `[service/account/federated]`
- Conditions: [present/missing] — [assessment]

**Permission analysis:**

| Statement | Actions | Resources | Risk |
|---|---|---|---|
| Allow s3 access | `s3:GetObject`, `s3:PutObject` | `arn:aws:s3:::my-bucket/*` | LOW — scoped correctly |
| Allow SSM | `ssm:GetParameters` | `arn:aws:ssm:*:*:parameter/*` | MEDIUM — resource not scoped |

**Issues:**
- [Issue 1: specific finding with line reference]
- [Issue 2: ...]

**Recommended changes:**
```yaml
# Show the corrected IAM statement(s)
```

---

## Summary

| Role | Permission boundary | Wildcard resources | Wildcard actions | Overall risk |
|---|---|---|---|---|
| AppTaskRole | PRESENT | No | No | LOW |
| DeployRole | MISSING | Yes | No | HIGH |

## Privilege escalation paths

Check if any combination of allowed actions could let this role escalate its own privileges:

Dangerous combinations:
- `iam:CreateRole` + `iam:AttachRolePolicy` → can create admin role
- `iam:PassRole` with broad resource → can pass a more-privileged role
- `sts:AssumeRole` with `Resource: "*"` → can assume any role it can reach

Flag any escalation paths found.

## OWASP Cloud Top 10 mapping

Map findings to relevant OWASP Cloud Security Top 10 items:
- C1: Insufficient Identity, Credential, and Access Management
- C2: Insecure Interfaces and APIs
- C5: Insufficient Logging and Monitoring
