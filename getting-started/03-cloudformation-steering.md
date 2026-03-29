# Step 3: CloudFormation Steering

## What to put in CFN steering files

The goal is to encode your org's opinions so that every template Kiro generates:
1. Passes cfn-lint with no warnings
2. Has all required tags
3. Uses your naming conventions
4. Follows your security baseline (encryption, no public exposure by default)
5. Doesn't need the standard "please add tags / please encrypt this" review comments

## Copy the starter steering files

```bash
# From this repo, copy the CloudFormation steering to your infra repo
cp steering/cloudformation/*.md /path/to/your/infra-repo/.kiro/steering/
```

Then **edit each file** to replace the placeholder values with your org's actual values:
- Account IDs
- Region preferences
- Tag key names
- Naming prefixes
- VPC/subnet IDs used as defaults
- IAM role ARNs

## Starter steering files included in this repo

| File | Purpose |
|---|---|
| [cfn-structure.md](../steering/cloudformation/cfn-structure.md) | Template structure, metadata, transform patterns |
| [cfn-naming.md](../steering/cloudformation/cfn-naming.md) | Resource and stack naming conventions |
| [cfn-tagging.md](../steering/cloudformation/cfn-tagging.md) | Mandatory tags and tag propagation |
| [cfn-security.md](../steering/cloudformation/cfn-security.md) | Encryption, IAM, network security defaults |
| [cfn-parameters.md](../steering/cloudformation/cfn-parameters.md) | Parameter patterns, SSM references |
| [cfn-outputs.md](../steering/cloudformation/cfn-outputs.md) | Output conventions for cross-stack references |

## Test your steering is working

Open the Kiro chat and ask:

```
Create a CloudFormation template for an S3 bucket that will store application logs.
```

With no steering, you'll get a basic template. With the CFN steering files loaded, you should get:
- Your required tags already present
- Encryption enabled by default
- Your naming convention applied
- Lifecycle rules if your steering specifies them
- `DeletionPolicy` set per your policy

If it doesn't include those, check that the steering files are in `.kiro/steering/` and re-open the project.

## Tips for iterating on steering files

1. **Start with what causes the most review churn.** If every PR review includes "please add tags", write a tagging steering file first.
2. **Use real examples from your codebase.** Copy a well-written existing template as the example in the steering file.
3. **Version them with your repo.** Steering files are code. Review them in PRs like any other infrastructure change.
4. **Prune aggressively.** A steering file with 50 rules is worse than one with 10 that are always followed. Quality over quantity.

## Next step

[04 - Configure MCP servers](./04-mcp-servers-setup.md)
