---
includeBy:
  - "**/*.yaml"
  - "**/*.yml"
  - "**/cloudformation/**"
---

# CloudFormation Outputs and Cross-Stack References

## When to use Outputs

Output every value that another stack might need. Be generous — it is easier to add an output now than to modify the stack later.

Standard outputs for every stack type:

**Networking stacks:** VPC ID, subnet IDs (by tier), security group IDs, NAT Gateway IDs
**Compute stacks:** ECS cluster name, ECS service name, ASG name, ALB DNS name, ALB ARN
**Data stacks:** RDS endpoint, RDS port, RDS secret ARN, DynamoDB table name/ARN
**IAM stacks:** Role ARNs (not role names — ARNs are cross-account safe)

## Output format

```yaml
Outputs:
  VpcId:
    Description: "VPC ID for use by workload stacks"
    Value: !Ref VPC
    Export:
      Name: !Sub "${AWS::StackName}-vpc-id"

  PrivateSubnetIds:
    Description: "Comma-separated list of private subnet IDs"
    Value: !Join [",", [!Ref PrivateSubnetA, !Ref PrivateSubnetB, !Ref PrivateSubnetC]]
    Export:
      Name: !Sub "${AWS::StackName}-private-subnet-ids"
```

Always include a `Description`. Always use `!Sub "${AWS::StackName}-..."` for export names — this ensures uniqueness per region.

## Export naming convention

```
{stack-name}-{resource-type}-{identifier}

platform-network-prod-vpc-id
platform-network-prod-private-subnet-ids
platform-network-prod-public-subnet-ids
order-service-prod-alb-dns-name
order-service-prod-ecs-service-name
```

## Cross-stack imports

Two patterns are acceptable. Prefer SSM for infrastructure values (more flexible), use `Fn::ImportValue` for tightly-coupled stacks.

### Pattern 1: SSM Parameter Store (preferred for infrastructure)

The producing stack writes to SSM, the consuming stack reads from SSM:

```yaml
# Producing stack writes:
VpcIdParam:
  Type: AWS::SSM::Parameter
  Properties:
    Name: !Sub "/platform/network/${Environment}/vpc-id"
    Type: String
    Value: !Ref VPC

# Consuming stack reads at deploy time (no runtime dependency):
Parameters:
  VpcId:
    Type: AWS::SSM::Parameter::Value<AWS::EC2::VPC::Id>
    Default: /platform/network/prod/vpc-id
```

**Advantage:** Stacks are decoupled. You can delete and recreate the producing stack without breaking the consuming stack (as long as SSM is repopulated).

### Pattern 2: Fn::ImportValue (for tightly-coupled stacks)

```yaml
Resources:
  MyService:
    Properties:
      VpcId: !ImportValue "platform-network-prod-vpc-id"
      SubnetIds: !Split [",", !ImportValue "platform-network-prod-private-subnet-ids"]
```

**Caution:** CloudFormation prevents deletion of any stack that has active exports being imported. If you use this pattern, you cannot delete the producing stack while any consuming stack exists.

## What NOT to output

- Don't output things that change frequently (e.g., task definition revision numbers)
- Don't output internal implementation details that callers shouldn't depend on
- Don't output sensitive values (passwords, keys) — use Secrets Manager ARNs instead, never the secret itself
