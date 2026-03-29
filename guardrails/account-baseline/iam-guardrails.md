---
# Always active — IAM guardrails apply to all files
---

# IAM Guardrails

## Permission boundary — mandatory on all roles

Every IAM role generated for any purpose MUST include a permission boundary. No exceptions.

```yaml
MyRole:
  Type: AWS::IAM::Role
  Properties:
    PermissionsBoundary: !Sub "arn:aws:iam::${AWS::AccountId}:policy/OrgPermissionBoundary"
    RoleName: !Sub "${Team}-${Environment}-${Purpose}-role"
    AssumeRolePolicyDocument:
      Version: "2012-10-17"
      Statement:
        - Effect: Allow
          Principal:
            Service: [appropriate service].amazonaws.com
          Action: sts:AssumeRole
```

## Service-linked roles

Service-linked roles (created by `AWS::IAM::ServiceLinkedRole`) are exempt from the permission boundary requirement as they're managed by AWS.

## ECS task roles

ECS tasks need two roles — generate both with boundaries:

```yaml
ECSTaskExecutionRole:
  Type: AWS::IAM::Role
  Properties:
    PermissionsBoundary: !Sub "arn:aws:iam::${AWS::AccountId}:policy/OrgPermissionBoundary"
    AssumeRolePolicyDocument:
      Statement:
        - Effect: Allow
          Principal:
            Service: ecs-tasks.amazonaws.com
          Action: sts:AssumeRole
    ManagedPolicyArns:
      - arn:aws:iam::aws:policy/service-role/AmazonECSTaskExecutionRolePolicy

ECSTaskRole:
  Type: AWS::IAM::Role
  Properties:
    PermissionsBoundary: !Sub "arn:aws:iam::${AWS::AccountId}:policy/OrgPermissionBoundary"
    AssumeRolePolicyDocument:
      Statement:
        - Effect: Allow
          Principal:
            Service: ecs-tasks.amazonaws.com
          Action: sts:AssumeRole
    Policies:
      - PolicyName: AppPermissions
        PolicyDocument:
          Statement:
            # Only specific permissions the app needs
```

## Lambda execution roles

```yaml
LambdaExecutionRole:
  Type: AWS::IAM::Role
  Properties:
    PermissionsBoundary: !Sub "arn:aws:iam::${AWS::AccountId}:policy/OrgPermissionBoundary"
    AssumeRolePolicyDocument:
      Statement:
        - Effect: Allow
          Principal:
            Service: lambda.amazonaws.com
          Action: sts:AssumeRole
    ManagedPolicyArns:
      - arn:aws:iam::aws:policy/service-role/AWSLambdaVPCAccessExecutionRole
    Policies:
      - PolicyName: FunctionPermissions
        PolicyDocument:
          Statement:
            - Effect: Allow
              Action:
                - logs:CreateLogGroup
                - logs:CreateLogStream
                - logs:PutLogEvents
              Resource: !Sub "arn:aws:logs:${AWS::Region}:${AWS::AccountId}:log-group:/aws/lambda/*"
```

## GitHub Actions OIDC deployment roles

For CI/CD deployment roles using OIDC federation:

```yaml
GitHubActionsRole:
  Type: AWS::IAM::Role
  Properties:
    PermissionsBoundary: !Sub "arn:aws:iam::${AWS::AccountId}:policy/OrgPermissionBoundary"
    RoleName: !Sub "GitHubActions-Deploy-${Environment}"
    AssumeRolePolicyDocument:
      Statement:
        - Effect: Allow
          Principal:
            Federated: !Sub "arn:aws:iam::${AWS::AccountId}:oidc-provider/token.actions.githubusercontent.com"
          Action: sts:AssumeRoleWithWebIdentity
          Condition:
            StringEquals:
              "token.actions.githubusercontent.com:aud": "sts.amazonaws.com"
            StringLike:
              "token.actions.githubusercontent.com:sub": "repo:your-org/*:ref:refs/heads/main"
```

## What the OrgPermissionBoundary prevents

The boundary policy denies these actions even if a role's identity policy allows them:
- Creating IAM roles or policies without the boundary
- Modifying the baseline security stack
- Disabling CloudTrail or Config
- Accessing the management account
- Escalating to admin via `iam:PassRole` to admin roles
- Creating or accessing IAM users (no long-lived credentials in workloads)
