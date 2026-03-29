# About This Repository

This is the **kiro-for-cloud-teams** starter kit — a guide and template library for Cloud Platform and SRE teams adopting Kiro.

## What this repo contains

This repo IS the guide. It contains:
- Step-by-step getting-started docs in `getting-started/`
- Steering file templates in `steering/`
- Guardrail templates in `guardrails/`
- MCP server configuration in `mcp-servers/`
- Reusable skill definitions in `skills/`
- Worked CloudFormation examples in `examples/`

## How to use this repo

When helping someone with this repo, prioritise:

1. **Explaining how to adopt the patterns** — most questions are about how to copy/adapt these files into a real infrastructure repo
2. **Customising the templates** — help the user replace placeholder values with their org's actual values
3. **Extending the steering** — help write new steering files for their specific conventions

## Key conventions used throughout

- CloudFormation templates use `Environment`, `Team`, `CostCentre` as standard parameters
- All IAM roles require `OrgPermissionBoundary` — replace the ARN with your org's actual boundary
- SSM Parameter paths follow `/platform/network/{environment}/` for shared infrastructure outputs
- Naming convention: `{team}-{environment}-{region}-{purpose}` for physical resources

## What to customise before using in a real org

Search for these placeholders and replace them:

| Placeholder | What to replace with |
|---|---|
| `OrgPermissionBoundary` | Your actual IAM permission boundary policy name |
| `CC-[0-9]+` | Your cost centre code format |
| `your-org` | Your GitHub organisation name |
| `eu-west-1` | Your primary AWS region |
| `log-archive-${AWS::AccountId}-${AWS::Region}` | Your centralised log bucket name |
| `/platform/network/` | Your SSM parameter path prefix |
