---
includeBy:
  - "**/*.yaml"
  - "**/*.yml"
  - "**/cloudformation/**"
  - "**/cfn/**"
---

# CloudFormation Tagging Standards

## Mandatory tags

Every resource that supports tagging MUST have all of these tags:

| Tag key | Value pattern | Example | Notes |
|---|---|---|---|
| `Environment` | `dev`, `staging`, `prod` | `prod` | From `!Ref Environment` parameter |
| `Team` | lowercase, hyphens only | `platform-engineering` | From `!Ref Team` parameter |
| `CostCentre` | CC-NNNN | `CC-1042` | Your finance cost centre code |
| `ManagedBy` | `cloudformation` | `cloudformation` | Always literal |
| `StackName` | | `!Ref AWS::StackName` | Auto-populated from stack |
| `Application` | lowercase, hyphens | `order-service` | From parameter or hardcoded |

## How to apply tags

Use a `Tags` section on every resource. Use `!Sub` or `!Ref` to reference parameters — never hardcode environment names:

```yaml
Resources:
  MyBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: !Sub "${Team}-${Environment}-${AWS::Region}-logs"
      Tags:
        - Key: Environment
          Value: !Ref Environment
        - Key: Team
          Value: !Ref Team
        - Key: CostCentre
          Value: !Ref CostCentre
        - Key: ManagedBy
          Value: cloudformation
        - Key: StackName
          Value: !Ref AWS::StackName
        - Key: Application
          Value: !Ref Application
```

## Tag propagation for Auto Scaling

ASGs do not propagate tags to EC2 instances by default. Always set `PropagateAtLaunch: true`:

```yaml
AutoScalingGroup:
  Type: AWS::AutoScaling::AutoScalingGroup
  Properties:
    Tags:
      - Key: Environment
        Value: !Ref Environment
        PropagateAtLaunch: true
      - Key: Team
        Value: !Ref Team
        PropagateAtLaunch: true
```

## Tag propagation for ECS

ECS tasks inherit tags from the service if you enable it:

```yaml
MyECSService:
  Type: AWS::ECS::Service
  Properties:
    EnableECSManagedTags: true
    PropagateTags: SERVICE
```

## Tag via CloudFormation stack tags

For tags that apply to all resources in a stack, use stack-level tags in your deployment:

```yaml
# GitHub Actions / deployment script
aws cloudformation deploy \
  --template-file template.yaml \
  --tags Environment=prod Team=platform CostCentre=CC-1042 ManagedBy=cloudformation
```

Stack-level tags propagate to all supported resources automatically. Use them for the common tags and resource-level `Tags:` only for resource-specific tags.

## Validation

Before deploging, check tags with:

```bash
# Using cfn-lint custom rule (see .cfnlintrc in this repo)
cfn-lint template.yaml --include-checks I

# Or using checkov
checkov -f template.yaml --check CKV_AWS_RESOURCE_TAGS
```

## Cost allocation

Tags `CostCentre`, `Team`, and `Environment` are activated for cost allocation in AWS Cost Explorer. Untagged resources appear under "No Tag" and will be flagged in the monthly cost review.
