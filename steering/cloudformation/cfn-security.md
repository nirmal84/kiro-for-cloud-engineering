---
includeBy:
  - "**/*.yaml"
  - "**/*.yml"
  - "**/cloudformation/**"
---

# CloudFormation Security Standards

## Encryption: always on by default

Every storage resource must be encrypted. There are no exceptions for any environment.

### S3

```yaml
MyBucket:
  Type: AWS::S3::Bucket
  Properties:
    BucketEncryption:
      ServerSideEncryptionConfiguration:
        - ServerSideEncryptionByDefault:
            SSEAlgorithm: aws:kms
            KMSMasterKeyID: !Ref KMSKeyArn   # Use CMK, not AWS-managed key for prod
    PublicAccessBlockConfiguration:
      BlockPublicAcls: true
      BlockPublicPolicy: true
      IgnorePublicAcls: true
      RestrictPublicBuckets: true
    VersioningConfiguration:
      Status: Enabled   # Required for all prod buckets
```

### RDS

```yaml
MyDatabase:
  Type: AWS::RDS::DBInstance
  Properties:
    StorageEncrypted: true
    KmsKeyId: !Ref KMSKeyArn
    MultiAZ: !If [IsProd, true, false]
    PubliclyAccessible: false
    DeletionProtection: !If [IsProd, true, false]
    EnableCloudwatchLogsExports:
      - postgresql    # or mysql, error, audit depending on engine
```

### EBS volumes

```yaml
MyInstance:
  Type: AWS::EC2::Instance
  Properties:
    BlockDeviceMappings:
      - DeviceName: /dev/xvda
        Ebs:
          Encrypted: true
          KmsKeyId: !Ref KMSKeyArn
          VolumeType: gp3
```

### DynamoDB

```yaml
MyTable:
  Type: AWS::DynamoDB::Table
  Properties:
    SSESpecification:
      SSEEnabled: true
      SSEType: KMS
      KMSMasterKeyId: !Ref KMSKeyArn
    PointInTimeRecoverySpecification:
      PointInTimeRecoveryEnabled: true
```

## IAM: principle of least privilege

### Permission boundaries on every role

```yaml
MyRole:
  Type: AWS::IAM::Role
  Properties:
    PermissionsBoundary: !Sub "arn:aws:iam::${AWS::AccountId}:policy/OrgPermissionBoundary"
    AssumeRolePolicyDocument:
      Version: "2012-10-17"
      Statement:
        - Effect: Allow
          Principal:
            Service: lambda.amazonaws.com
          Action: sts:AssumeRole
```

### No wildcard resources in prod

```yaml
# WRONG - never do this
MyPolicy:
  Type: AWS::IAM::Policy
  Properties:
    PolicyDocument:
      Statement:
        - Effect: Allow
          Action: "s3:*"
          Resource: "*"

# CORRECT - scope to specific resource
MyPolicy:
  Type: AWS::IAM::Policy
  Properties:
    PolicyDocument:
      Statement:
        - Effect: Allow
          Action:
            - s3:GetObject
            - s3:PutObject
          Resource: !Sub "arn:aws:s3:::${MyBucket}/*"
```

### No inline policies on users

Never generate `AWS::IAM::User` with inline policies. Use roles and groups.

## Networking: private by default

```yaml
# Load balancers
MyALB:
  Type: AWS::ElasticLoadBalancingV2::LoadBalancer
  Properties:
    Scheme: !If [IsPublicFacing, internet-facing, internal]
    # Default to internal — be explicit when internet-facing

# Security groups: no 0.0.0.0/0 ingress except on port 443 for public ALBs
MySecurityGroup:
  Type: AWS::EC2::SecurityGroup
  Properties:
    SecurityGroupIngress:
      # WRONG: never add this except for public-facing HTTPS
      # - IpProtocol: "-1"
      #   CidrIp: "0.0.0.0/0"
      - IpProtocol: tcp
        FromPort: 443
        ToPort: 443
        CidrIp: "0.0.0.0/0"   # Only acceptable for public-facing HTTPS endpoints
```

## Logging: always enabled

```yaml
# CloudTrail (account-level — in the baseline stack, not per service)
# Per-service: enable access logging on all ALBs and S3 buckets

MyALB:
  Type: AWS::ElasticLoadBalancingV2::LoadBalancer
  Properties:
    LoadBalancerAttributes:
      - Key: access_logs.s3.enabled
        Value: "true"
      - Key: access_logs.s3.bucket
        Value: !Sub "log-archive-${AWS::AccountId}-${AWS::Region}"
```

## Secrets: never hardcode

```yaml
# WRONG
MyDatabase:
  Properties:
    MasterUserPassword: "Passw0rd123"

# CORRECT - use Secrets Manager
MySecret:
  Type: AWS::SecretsManager::Secret
  Properties:
    GenerateSecretString:
      SecretStringTemplate: '{"username": "admin"}'
      GenerateStringKey: password
      PasswordLength: 32
      ExcludeCharacters: '"@/\'

MyDatabase:
  Properties:
    MasterUserPassword: !Sub "{{resolve:secretsmanager:${MySecret}:SecretString:password}}"
```

## Security checklist before deploying

Before deploying any template, verify:

- [ ] All storage resources have encryption enabled
- [ ] All IAM roles have permission boundaries
- [ ] No `"Resource": "*"` with broad actions
- [ ] No hardcoded credentials or account IDs
- [ ] No security groups with `0.0.0.0/0` ingress except port 443 on public ALBs
- [ ] All resources have required tags
- [ ] Logging enabled on load balancers, S3, RDS
- [ ] `PubliclyAccessible: false` on RDS
- [ ] `BlockPublicAcls: true` and related settings on S3
