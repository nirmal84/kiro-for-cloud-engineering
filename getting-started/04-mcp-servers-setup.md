# Step 4: Configure MCP Servers

## What are MCP servers?

MCP (Model Context Protocol) servers give Kiro tools it can call during a conversation — live AWS API calls, documentation lookups, linting, cost estimates. Without them, Kiro can only reason about code it can see. With them, it can check what's actually deployed, look up current pricing, or validate a template before you ever run `aws cloudformation deploy`.

## Configuring MCP servers in Kiro

MCP servers are configured in `.kiro/settings.json`:

```json
{
  "mcpServers": {
    "server-name": {
      "command": "npx",
      "args": ["-y", "@package/mcp-server"],
      "env": {
        "AWS_PROFILE": "my-profile"
      }
    }
  }
}
```

Kiro starts the MCP server as a subprocess when the project opens. The server stays running until you close the project.

## Recommended MCP servers for cloud teams

See [mcp-servers/README.md](../mcp-servers/README.md) for the full list with install instructions.

**Start with these three:**

### 1. AWS Documentation MCP

Gives Kiro access to current AWS service documentation and CloudFormation resource specifications.

```json
"aws-docs": {
  "command": "uvx",
  "args": ["awslabs.aws-documentation-mcp-server@latest"],
  "env": { "FASTMCP_LOG_LEVEL": "ERROR" }
}
```

**What it unlocks:** Kiro can look up the exact CloudFormation resource schema, check supported properties, and reference current service limits without hallucinating outdated API shapes.

### 2. AWS CloudFormation MCP

Direct CloudFormation API access — describe stacks, get stack events, list resources.

```json
"cloudformation": {
  "command": "uvx",
  "args": ["awslabs.aws-cloudformation-mcp-server@latest"],
  "env": {
    "AWS_PROFILE": "${env:AWS_PROFILE}",
    "AWS_REGION": "eu-west-1"
  }
}
```

**What it unlocks:** Ask Kiro "what resources are in the prod-network stack?" or "why did this stack fail?" without leaving the editor.

### 3. AWS Cost Explorer MCP

Cost and usage data via the Cost Explorer API.

```json
"cost-explorer": {
  "command": "uvx",
  "args": ["awslabs.aws-cost-analysis-mcp-server@latest"],
  "env": {
    "AWS_PROFILE": "${env:AWS_PROFILE}"
  }
}
```

**What it unlocks:** Ask "how much is the data transfer from our eu-west-1 ECS cluster costing per month?" before you re-architect it.

## The full recommended stack

See [mcp-servers/settings-template.json](../mcp-servers/settings-template.json) for a copy-paste `.kiro/settings.json` with all recommended servers.

| MCP Server | Use case |
|---|---|
| `aws-documentation` | CFN resource schemas, service docs |
| `aws-cloudformation` | Live stack inspection, drift detection |
| `aws-cost-analysis` | Cost attribution, rightsizing analysis |
| `aws-security-hub` | Finding review, compliance posture |
| `github` | PR creation, issue tracking from chat |
| `aws-cdk` | CDK construct lookups, L2 patterns |
| `kubernetes` | EKS workload inspection |

## Security note: MCP servers and credentials

MCP servers run with your local AWS credentials. Apply least privilege:

- Use a read-only AWS profile for documentation/inspection servers
- Use a scoped profile for any server that can make changes
- Never put raw credentials in `settings.json` — use `${env:VAR}` references
- Check `.kiro/settings.json` into git, but ensure it contains no secrets

```json
"env": {
  "AWS_PROFILE": "${env:AWS_PROFILE}",
  "AWS_REGION": "${env:AWS_DEFAULT_REGION}"
}
```

## Next step

[05 - Build central guardrails](./05-central-guardrails.md)
