---
includeBy:
  - "**/*.yaml"
  - "**/*.yml"
  - "**/cloudformation/**"
---

# CloudFormation Naming Conventions

## Stack names

```
{service}-{component}-{environment}
```

Examples:
- `platform-network-prod`
- `ecommerce-api-staging`
- `data-warehouse-prod`

## Logical resource IDs (inside the template)

Use PascalCase. Be descriptive. Include the resource type suffix:

```yaml
Resources:
  AppServerSecurityGroup:      # Good: type suffix, descriptive
  AppSG:                       # Bad: abbreviation, unclear

  OrderServiceTaskDefinition:  # Good
  TaskDef:                     # Bad

  DatabaseSubnetGroup:         # Good
  DBSubGroup:                  # Bad
```

## Physical resource names (the `Name:` property)

```
{team}-{environment}-{region}-{purpose}
```

Use `!Sub` to build names from parameters:

```yaml
MyBucket:
  Type: AWS::S3::Bucket
  Properties:
    BucketName: !Sub "${Team}-${Environment}-${AWS::Region}-${Purpose}"
    # Example: platform-prod-eu-west-1-artifacts

MyVPC:
  Type: AWS::EC2::VPC
  Properties:
    Tags:
      - Key: Name
        Value: !Sub "${Team}-${Environment}-${AWS::Region}-vpc"
        # Example: platform-prod-eu-west-1-vpc

MyRole:
  Type: AWS::IAM::Role
  Properties:
    RoleName: !Sub "${Team}-${Environment}-${Purpose}-role"
    # Example: platform-prod-deploy-role
```

## Resource-specific naming patterns

### IAM roles and policies
```
{team}-{environment}-{function}-role
{team}-{environment}-{function}-policy

platform-prod-ecs-task-role
platform-prod-ecs-task-policy
```

### Security groups
```
{service}-{environment}-{tier}-sg

order-service-prod-app-sg
order-service-prod-db-sg
```

### Subnets
```
{team}-{environment}-{az}-{tier}-subnet

platform-prod-euw1a-private-subnet
platform-prod-euw1b-private-subnet
platform-prod-euw1a-public-subnet
```

### Parameter Store paths

Use hierarchical paths:

```
/{environment}/{service}/{parameter}

/prod/order-service/database-url
/prod/order-service/api-key
/staging/order-service/database-url
```

### Secrets Manager names

```
{environment}/{service}/{secret-name}

prod/order-service/db-credentials
prod/order-service/payment-api-key
```

### CloudWatch Log Groups

```
/aws/{service-type}/{team}-{environment}-{service}
/app/{team}/{environment}/{service}

/aws/ecs/platform-prod-api
/app/ecommerce/prod/order-service
```

## CloudFormation export names

For `Outputs` used as cross-stack references:

```
{stack-name}-{resource-type}-{identifier}

platform-network-prod-vpc-id
platform-network-prod-private-subnet-ids
platform-network-prod-public-subnet-ids
```

```yaml
Outputs:
  VpcId:
    Value: !Ref MyVPC
    Export:
      Name: !Sub "${AWS::StackName}-vpc-id"

  PrivateSubnetIds:
    Value: !Join [",", [!Ref PrivateSubnetA, !Ref PrivateSubnetB]]
    Export:
      Name: !Sub "${AWS::StackName}-private-subnet-ids"
```

## Do not use

- Underscores in resource names (use hyphens)
- Camel case in physical names (`myBucket` → `my-bucket`)
- Generic names (`MyBucket`, `MyRole`, `SecurityGroup1`)
- Names that don't include the environment (prevents name collisions across envs)
- Names longer than 64 characters (IAM role limit is 64 chars)
