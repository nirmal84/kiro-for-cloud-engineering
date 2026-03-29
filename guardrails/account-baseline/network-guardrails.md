---
# Always active — network guardrails apply globally
---

# Network Guardrails

## Default: private subnets only for workloads

Unless explicitly deploying a public-facing load balancer, all compute resources go into private subnets. This includes:
- ECS Fargate tasks
- EC2 instances
- Lambda functions in a VPC
- RDS, ElastiCache, OpenSearch

```yaml
# ECS — always DISABLED for public IP
NetworkConfiguration:
  AwsvpcConfiguration:
    AssignPublicIp: DISABLED    # Never ENABLED for app tasks
    Subnets: !Ref PrivateSubnetIds

# RDS — never in public subnet
MyDatabase:
  Properties:
    DBSubnetGroupName: !Ref DataSubnetGroup   # data tier, not public
    PubliclyAccessible: false
```

## No SSH or RDP open to the internet

Replace SSH/RDP with AWS Systems Manager Session Manager. This requires the SSM agent (pre-installed on Amazon Linux 2/2023 and Windows Server AMIs) and the SSM endpoints in the VPC.

```yaml
# NEVER generate this:
SecurityGroupIngress:
  - IpProtocol: tcp
    FromPort: 22
    ToPort: 22
    CidrIp: "0.0.0.0/0"

# Instead, use SSM Session Manager:
# No inbound rule needed at all for SSM connectivity
# ECS: use ECS Exec (EnableExecuteCommand: true on the service)
# EC2: ensure SSM agent is running and SSM VPC endpoints exist
```

To start a session via SSM:
```bash
aws ssm start-session --target i-0123456789abcdef0
# Or for ECS:
aws ecs execute-command --cluster my-cluster --task task-id --container app --command "/bin/sh" --interactive
```

## Security group rules: reference groups, not CIDRs

For internal traffic, reference security group IDs rather than CIDR blocks. This is more maintainable and scales with dynamic IP changes.

```yaml
# PREFER: reference the ALB security group
ServiceSecurityGroup:
  SecurityGroupIngress:
    - IpProtocol: tcp
      FromPort: 8080
      ToPort: 8080
      SourceSecurityGroupId: !Ref AlbSecurityGroup
      Description: "From ALB only"

# AVOID (unless cross-VPC where you must use CIDR):
ServiceSecurityGroup:
  SecurityGroupIngress:
    - IpProtocol: tcp
      FromPort: 8080
      ToPort: 8080
      CidrIp: "10.0.0.0/8"    # Hard to trace, breaks if CIDR changes
```

## VPC endpoints: required for all private workloads

Private subnets cannot reach AWS services without either NAT Gateways or VPC endpoints. The baseline deploys standard endpoints. When adding a new service that needs to call AWS APIs, check that the relevant endpoint exists before relying on the NAT Gateway.

Required VPC endpoints (deployed by baseline):
- `com.amazonaws.{region}.s3` — Gateway (free)
- `com.amazonaws.{region}.dynamodb` — Gateway (free)
- `com.amazonaws.{region}.ecr.api` — Interface
- `com.amazonaws.{region}.ecr.dkr` — Interface
- `com.amazonaws.{region}.secretsmanager` — Interface
- `com.amazonaws.{region}.ssm` — Interface
- `com.amazonaws.{region}.ssmmessages` — Interface
- `com.amazonaws.{region}.logs` — Interface

If your workload calls other AWS services (e.g., SQS, SNS, Kinesis), add their endpoints to the VPC baseline rather than routing traffic over the NAT Gateway.

## VPC CIDR planning

Standard CIDR allocation (replace with your org's actual scheme):

| Account type | CIDR range | Notes |
|---|---|---|
| Production | 10.0.0.0/16 – 10.15.255.255/16 | 16 prod VPCs |
| Staging | 10.16.0.0/16 – 10.31.255.255/16 | 16 staging VPCs |
| Dev/sandbox | 10.32.0.0/16 – 10.63.255.255/16 | 32 dev VPCs |
| Shared services | 10.64.0.0/16 | Central services |
| Network account | 10.65.0.0/16 | TGW, DNS, inspection |

**Never reuse CIDRs.** Overlapping CIDRs cannot be routed via Transit Gateway.

## No default VPC usage

The baseline deletes the default VPC in every account. Never deploy workloads to the default VPC. If you see a template referencing `default` for VPC or subnet, replace it with the org-standard SSM parameter reference.

## Network ACLs

The baseline creates restrictive NACLs on data subnets. Do not modify or delete these NACLs in service stacks — they are managed by the baseline.

For application and public subnets, rely on security groups (stateful) rather than NACLs (stateless) for fine-grained control.
