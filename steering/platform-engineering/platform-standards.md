---
includeBy:
  - "**/platform/**"
  - "**/account-vending/**"
  - "**/org/**"
  - "**/landing-zone/**"
---

# Platform Engineering Standards

## Account structure

Your AWS Organization follows this account structure:

```
Root (management account - NEVER deploy workloads here)
├── Security OU
│   ├── Log Archive account    (centralised CloudTrail, Config, VPC Flow Logs)
│   └── Security Tooling       (GuardDuty master, Security Hub master, Macie)
├── Infrastructure OU
│   ├── Network account        (Transit Gateway, shared VPCs, DNS)
│   ├── Shared Services        (ECR, artifact buckets, internal tools)
│   └── Sandbox accounts       (per-team, auto-expire after 90 days)
├── Workloads OU
│   ├── Non-Prod
│   │   ├── Dev accounts (per team or shared)
│   │   └── Staging accounts
│   └── Prod
│       └── Production accounts (per service domain)
```

## Account baseline requirements

Every new account MUST have these deployed before any workloads:

1. **Baseline security stack** — CloudTrail, Config, GuardDuty enrollment, Security Hub enrollment
2. **Network baseline** — VPC with private/public subnets, Transit Gateway attachment, VPC endpoints for S3/ECR/Secrets Manager
3. **IAM baseline** — Permission boundary policy, break-glass role, read-only audit role
4. **Log forwarding** — CloudWatch Logs forwarding to centralized log archive
5. **Budget alerts** — Default budget alarm at 80% of monthly allocation

## Account vending

New accounts are provisioned via the account-vending pipeline, not manually. To request a new account:

1. Submit PR to the account-vending repo with the account definition YAML
2. Platform team reviews and approves
3. Pipeline provisions account, applies baseline, creates SSO assignments

Account definition template: [see examples/account-vending/](../../examples/account-vending/)

## Cross-account access patterns

```yaml
# Service-to-service: assume role via resource-based policy
# Application accounts assume roles in shared services accounts

# Pattern: trust policy in the target account
TrustPolicy:
  Statement:
    - Effect: Allow
      Principal:
        AWS: !Sub "arn:aws:iam::${SourceAccountId}:role/${SourceRoleName}"
      Action: sts:AssumeRole
      Condition:
        StringEquals:
          sts:ExternalId: !Ref ExternalId  # Required for third-party access
```

## Transit Gateway and networking

- All workload VPCs attach to the central Transit Gateway
- Route tables are managed centrally in the Network account
- No VPC peering between workload accounts — use TGW
- DNS: centralized Route53 resolver in Network account, forwarding rules pushed to all accounts

## Shared services patterns

### ECR (container images)
```
# Registry URI pattern:
{shared-services-account-id}.dkr.ecr.{region}.amazonaws.com/{team}/{service}:{tag}

123456789.dkr.ecr.eu-west-1.amazonaws.com/platform/nginx-proxy:v1.2.3
```

### Artifact S3
```
# Bucket pattern in Shared Services account:
artifacts-{shared-services-account-id}-{region}

# Access: workload accounts assume ArtifactReadRole to pull
```

## IAM permission boundaries

All platform-provisioned accounts include a permission boundary policy called `OrgPermissionBoundary`. All roles and users must have this boundary attached.

The boundary prevents:
- Creating IAM entities without the boundary themselves
- Accessing the management account
- Disabling CloudTrail or Config
- Modifying the baseline security stack

```yaml
# Always reference this in role creation:
MyRole:
  Type: AWS::IAM::Role
  Properties:
    PermissionsBoundary: !Sub "arn:aws:iam::${AWS::AccountId}:policy/OrgPermissionBoundary"
```
