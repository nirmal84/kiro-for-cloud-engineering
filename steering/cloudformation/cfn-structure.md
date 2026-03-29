---
includeBy:
  - "**/*.yaml"
  - "**/*.yml"
  - "**/cloudformation/**"
  - "**/cfn/**"
  - "**/templates/**"
---

# CloudFormation Template Structure

## Required template sections

All CloudFormation templates must include these sections in this order:

```yaml
AWSTemplateFormatVersion: "2010-09-09"
Description: "[Service/Component] - [What it creates] - [Environment]"

Metadata:
  AWS::CloudFormation::Interface:
    ParameterGroups:
      - Label:
          default: Environment Configuration
        Parameters:
          - Environment
          - Team
    ParameterLabels:
      Environment:
        default: Deployment Environment

Parameters:
  # ... parameters here

Conditions:
  # ... conditions here (if needed)

Resources:
  # ... resources here

Outputs:
  # ... outputs here
```

## Description format

```
"[Service] - [Brief description of what this creates] - Managed by [Team]"
```

Examples:
- `"Networking - VPC with 3-tier subnets for application workloads - Managed by Platform Team"`
- `"Data - RDS PostgreSQL cluster for order management service - Managed by Ecommerce Team"`

## Mandatory parameters

Every template must include:

```yaml
Parameters:
  Environment:
    Type: String
    AllowedValues:
      - dev
      - staging
      - prod
    Description: Deployment environment

  Team:
    Type: String
    Description: Owning team name (lowercase, no spaces)
    AllowedPattern: "^[a-z0-9-]+$"
```

## Transform for SAM/serverless

If using SAM transforms:

```yaml
AWSTemplateFormatVersion: "2010-09-09"
Transform: AWS::Serverless-2016-10-31
```

## Deletion policies

| Resource type | Non-prod | Prod |
|---|---|---|
| RDS, Aurora | `Delete` | `Retain` |
| S3 buckets | `Delete` | `Retain` |
| DynamoDB tables | `Delete` | `Retain` |
| ElasticSearch/OpenSearch | `Delete` | `Retain` |
| EFS | `Delete` | `Retain` |
| All others | `Delete` | `Delete` |

```yaml
MyDatabase:
  Type: AWS::RDS::DBInstance
  DeletionPolicy: !If [IsProd, Retain, Delete]
  UpdateReplacePolicy: !If [IsProd, Retain, Delete]
```

## Nested stacks

Use nested stacks for components over ~200 lines. The parent stack should be a thin orchestration layer.

```yaml
NetworkStack:
  Type: AWS::CloudFormation::Stack
  Properties:
    TemplateURL: !Sub "https://s3.amazonaws.com/${ArtifactBucket}/network.yaml"
    Parameters:
      Environment: !Ref Environment
```

## Anti-patterns to avoid

- Do not hardcode account IDs — use `!Sub ${AWS::AccountId}` or SSM parameters
- Do not hardcode region — use `!Sub ${AWS::Region}` or `!Ref AWS::Region`
- Do not create resources with names that don't include the `Environment` parameter
- Do not omit `Metadata` section — it's required for console readability
