---
# No includeBy — this is always active
---

# Security Guardrails

## Never generate these patterns

The following patterns must never appear in generated code, templates, or configurations:

### Hard-coded credentials

```
# These patterns MUST NOT appear in any file:
aws_access_key_id = AKIA...
aws_secret_access_key = ...
password = "..."
secret = "..."
api_key = "..."
```

If a secret is needed, use:
- AWS Secrets Manager: `{{resolve:secretsmanager:secret-name:SecretString:key}}`
- SSM SecureString: `{{resolve:ssm-secure:/path/to/param}}`
- IAM roles for service-to-service (no credentials at all)

### Over-permissive IAM

```yaml
# NEVER generate:
- Effect: Allow
  Action: "*"
  Resource: "*"

- Effect: Allow
  Action: "iam:*"
  Resource: "*"

- Effect: Allow
  Action: "s3:*"
  Resource: "*"
```

### Publicly accessible databases

```yaml
# NEVER generate:
PubliclyAccessible: true     # On RDS
AssociatePublicIpAddress: true   # On EC2 in private subnets
```

### Security groups open to world (except HTTPS)

```yaml
# NEVER generate (except port 443 on public ALBs):
SecurityGroupIngress:
  - IpProtocol: "-1"
    CidrIp: "0.0.0.0/0"
  - IpProtocol: tcp
    FromPort: 22
    ToPort: 22
    CidrIp: "0.0.0.0/0"    # SSH must never be open to world
  - IpProtocol: tcp
    FromPort: 3389
    ToPort: 3389
    CidrIp: "0.0.0.0/0"    # RDP must never be open to world
```

For SSH/RDP access, use SSM Session Manager instead.

### Disabling security controls

```yaml
# NEVER generate:
EnableDnsSupport: false
DisableApiTermination: false   # This one is OK to set
CloudTrailStatus: Stopped
ConfigRuleState: DELETED
```

## Required security controls

Every workload must have:

1. **VPC endpoints** for S3, ECR, Secrets Manager, SSM — prevents traffic going over the internet
2. **IMDSv2 only** on all EC2 instances and ECS task definitions
3. **CloudTrail enabled** in every account (managed by baseline, but verify)
4. **Config enabled** in every account (managed by baseline, but verify)
5. **GuardDuty enrolled** to org master

### IMDSv2 enforcement

```yaml
MyLaunchTemplate:
  Type: AWS::EC2::LaunchTemplate
  Properties:
    LaunchTemplateData:
      MetadataOptions:
        HttpTokens: required       # IMDSv2 only
        HttpPutResponseHopLimit: 1 # Prevent SSRF from reaching IMDS
        HttpEndpoint: enabled

# ECS task definition
MyTaskDefinition:
  Type: AWS::ECS::TaskDefinition
  Properties:
    # IMDSv2 for ECS tasks
    # Set via account-level setting or launch template on the ECS cluster
```

## Account-level security baseline

These are deployed once per account via the baseline stack. Do NOT recreate them in service stacks:

- CloudTrail (organisation trail, not per-account)
- AWS Config (rules deployed via org Config)
- GuardDuty (enrolled to org master)
- Security Hub (enrolled to org master, standards: AWS Foundational, CIS)
- IAM Access Analyzer
- Default VPC deleted (baseline removes it)

## Secrets rotation

All Secrets Manager secrets must have rotation enabled for prod:

```yaml
MySecret:
  Type: AWS::SecretsManager::Secret
  Properties:
    Name: !Sub "${Environment}/${Service}/db-credentials"

MySecretRotation:
  Type: AWS::SecretsManager::RotationSchedule
  Properties:
    SecretId: !Ref MySecret
    RotationLambdaARN: !Sub "arn:aws:lambda:${AWS::Region}:${AWS::AccountId}:function:SecretsManagerRotation"
    RotationRules:
      AutomaticallyAfterDays: 30
```

## Deployment security

- All deployments use OIDC federation from GitHub Actions
- Deployment roles follow least-privilege for the specific deployment action
- No human has persistent write access to prod (break-glass only, with audit trail)
- All prod deployments require a change ticket reference in the commit message
