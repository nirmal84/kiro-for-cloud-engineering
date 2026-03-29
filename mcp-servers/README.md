# MCP Servers for Cloud Teams

Recommended MCP servers for cloud platform and SRE workflows. All of these are official AWS Labs or well-maintained community servers.

## Prerequisites

```bash
# Install uv (Python package manager — used by most AWS MCP servers)
curl -LsSf https://astral.sh/uv/install.sh | sh

# Install Node.js (needed for some MCP servers)
brew install node   # macOS
```

## Complete settings.json

Copy [settings-template.json](./settings-template.json) to your repo's `.kiro/settings.json` and update the environment-specific values.

## Server reference

### AWS Documentation

**Package:** `awslabs.aws-documentation-mcp-server`

Fetches AWS service documentation, CloudFormation resource specifications, and API references.

```json
"aws-docs": {
  "command": "uvx",
  "args": ["awslabs.aws-documentation-mcp-server@latest"],
  "env": { "FASTMCP_LOG_LEVEL": "ERROR" }
}
```

**Use cases:**
- "What CloudFormation properties does AWS::ECS::Service support?"
- "What are the current limits for Lambda function size?"
- "Show me the VPC Flow Logs format"

---

### AWS CloudFormation

**Package:** `awslabs.aws-cloudformation-mcp-server`

Direct CloudFormation API access — describe stacks, list resources, get events.

```json
"cloudformation": {
  "command": "uvx",
  "args": ["awslabs.aws-cloudformation-mcp-server@latest"],
  "env": {
    "AWS_PROFILE": "${env:AWS_PROFILE}",
    "AWS_REGION": "${env:AWS_DEFAULT_REGION}"
  }
}
```

**Use cases:**
- "What is the current status of the platform-network-prod stack?"
- "List all resources in the order-service-staging stack"
- "Show me the last 10 events for the failing deployment"
- "Are there any stacks with drift?"

---

### AWS Cost Analysis

**Package:** `awslabs.aws-cost-analysis-mcp-server`

Cost Explorer API access for cost breakdown and anomaly detection.

```json
"cost-analysis": {
  "command": "uvx",
  "args": ["awslabs.aws-cost-analysis-mcp-server@latest"],
  "env": {
    "AWS_PROFILE": "${env:AWS_PROFILE}"
  }
}
```

**Use cases:**
- "What are the top 5 cost drivers in the prod account this month?"
- "Break down ECS costs by service tag"
- "Has there been a cost spike in the last 7 days?"
- "How much do our NAT Gateways cost per month?"

---

### AWS IAM Identity Center (SSO)

**Package:** `awslabs.aws-identity-center-mcp-server`

List permission sets, accounts, and access assignments.

```json
"iam-identity-center": {
  "command": "uvx",
  "args": ["awslabs.aws-identity-center-mcp-server@latest"],
  "env": {
    "AWS_PROFILE": "${env:AWS_MGMT_PROFILE}"
  }
}
```

**Use cases:**
- "Which accounts does the DevOps permission set have access to?"
- "Show me all access assignments for user@example.com"
- "What permission sets exist in our org?"

---

### AWS CDK

**Package:** `awslabs.cdk-mcp-server`

CDK construct library lookup and L2/L3 pattern recommendations.

```json
"cdk": {
  "command": "uvx",
  "args": ["awslabs.cdk-mcp-server@latest"],
  "env": { "FASTMCP_LOG_LEVEL": "ERROR" }
}
```

**Use cases:**
- "What CDK L2 construct should I use for an ECS Fargate service with ALB?"
- "Show me the props for the VPC construct"
- "What CDK patterns exist for serverless APIs?"

---

### Terraform (community)

**Package:** `hashicorp/terraform-mcp-server`

For teams using Terraform — registry lookup, module documentation.

```json
"terraform": {
  "command": "npx",
  "args": ["-y", "@hashicorp/terraform-mcp-server"]
}
```

**Use cases:**
- "What arguments does the aws_ecs_service resource take?"
- "Show me examples of the aws_rds_cluster module"

---

### GitHub

**Package:** `@modelcontextprotocol/server-github`

Create PRs, read issues, check CI status without leaving Kiro.

```json
"github": {
  "command": "npx",
  "args": ["-y", "@modelcontextprotocol/server-github"],
  "env": {
    "GITHUB_PERSONAL_ACCESS_TOKEN": "${env:GITHUB_TOKEN}"
  }
}
```

**Use cases:**
- "Create a PR for my current branch with a description of the changes"
- "What comments are on PR #234?"
- "Are CI checks passing on my branch?"
- "List open issues tagged 'platform-team'"

---

### Kubernetes / EKS (community)

**Package:** `mcp-server-kubernetes`

Kubectl-like access to your EKS clusters from chat.

```json
"kubernetes": {
  "command": "npx",
  "args": ["-y", "mcp-server-kubernetes"],
  "env": {
    "KUBECONFIG": "${env:HOME}/.kube/config"
  }
}
```

**Use cases:**
- "What pods are failing in the prod namespace?"
- "Show me the last 50 log lines from the order-service pod"
- "What is the current resource utilization of the prod cluster?"

---

## Minimal starter set

If you want to start with just the essentials:

```json
{
  "mcpServers": {
    "aws-docs": { ... },
    "cloudformation": { ... },
    "cost-analysis": { ... }
  }
}
```

Add more as your team identifies use cases. Too many MCP servers slow Kiro's startup and add noise to the tool selection.

## Security considerations

- Use read-only AWS profiles for inspection servers (`aws-docs`, `cloudformation`, `cost-analysis`)
- Never store credentials directly in `settings.json` — use `${env:VARIABLE}` references
- `settings.json` is checked into git — review it in code review like any infrastructure file
- GitHub token needs minimal scopes: `repo` (for PR creation), `read:org`
