---
includeBy:
  - "**/*.yaml"
  - "**/*.yml"
  - "**/cloudformation/**"
---

# CloudFormation Parameters Standards

## Always prefer SSM Parameter Store references

For values that change between environments (VPC IDs, subnet IDs, AMIs, certificate ARNs), use SSM Parameter Store references instead of hardcoding or passing as parameters:

```yaml
Parameters:
  # AVOID: forces caller to know the VPC ID
  VpcId:
    Type: AWS::EC2::VPC::Id

  # PREFER: resolve from SSM at deploy time
  VpcId:
    Type: AWS::SSM::Parameter::Value<AWS::EC2::VPC::Id>
    Default: /platform/network/prod/vpc-id
```

### SSM path conventions

Store environment infrastructure outputs in a consistent hierarchy:

```
/platform/network/{environment}/vpc-id
/platform/network/{environment}/private-subnet-ids
/platform/network/{environment}/public-subnet-ids
/platform/network/{environment}/certificate-arn
/platform/accounts/{environment}/id
/platform/ecr/{environment}/registry-url
```

Your baseline networking stack should export values to SSM:

```yaml
Outputs:
  VpcId:
    Value: !Ref VPC

# In a separate custom resource or SSM parameter resource:
VpcIdParameter:
  Type: AWS::SSM::Parameter
  Properties:
    Name: !Sub "/platform/network/${Environment}/vpc-id"
    Type: String
    Value: !Ref VPC
```

## Parameter types to use

```yaml
Parameters:
  Environment:
    Type: String
    AllowedValues: [dev, staging, prod]
    Description: Deployment environment

  Team:
    Type: String
    AllowedPattern: "^[a-z0-9-]+$"
    ConstraintDescription: Lowercase letters, numbers, and hyphens only

  # For AWS resource IDs — enables console dropdown
  SubnetId:
    Type: AWS::EC2::Subnet::Id

  # For lists of resource IDs
  SubnetIds:
    Type: List<AWS::EC2::Subnet::Id>

  # For AMI IDs — fetched from SSM
  LatestAmiId:
    Type: AWS::SSM::Parameter::Value<AWS::EC2::Image::Id>
    Default: /aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2

  # For sensitive values — shown as *** in console
  DatabasePassword:
    Type: String
    NoEcho: true   # Always use NoEcho for secrets
```

## Conditions from parameters

Define conditions once at the top, reference everywhere:

```yaml
Conditions:
  IsProd: !Equals [!Ref Environment, prod]
  IsNotProd: !Not [!Condition IsProd]
  IsMultiAz: !Or [!Condition IsProd, !Equals [!Ref Environment, staging]]
  EnableDeletionProtection: !Condition IsProd

# Use them:
Resources:
  MyDatabase:
    Properties:
      MultiAZ: !If [IsMultiAz, true, false]
      DeletionProtection: !If [EnableDeletionProtection, true, false]
```

## Parameter groups for console UX

Always include `AWS::CloudFormation::Interface` metadata:

```yaml
Metadata:
  AWS::CloudFormation::Interface:
    ParameterGroups:
      - Label:
          default: "Environment"
        Parameters:
          - Environment
          - Team
          - CostCentre
      - Label:
          default: "Networking"
        Parameters:
          - VpcId
          - SubnetIds
      - Label:
          default: "Compute"
        Parameters:
          - InstanceType
          - DesiredCapacity
    ParameterLabels:
      Environment:
        default: "Which environment are you deploying to?"
      CostCentre:
        default: "Finance cost centre code (CC-XXXX)"
```

## Default values policy

| Parameter | Default OK? | Notes |
|---|---|---|
| Environment | No | Must be explicit |
| Team | No | Must be explicit |
| CostCentre | No | Must be explicit |
| InstanceType | Yes | Use `t3.small` for non-prod defaults |
| DesiredCapacity | Yes | Use `1` for non-prod, no default for prod |
| SSM-resolved params | Yes | The SSM path is the default |
| Secrets | Never | Use Secrets Manager |
