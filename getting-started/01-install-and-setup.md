# Step 1: Install Kiro and Open Your First Project

## Install Kiro

Download Kiro from [kiro.dev](https://kiro.dev) and install it like any desktop app. It's built on VS Code so the interface is immediately familiar.

**Sign in with your AWS Builder ID** — this is free and required. If your org uses SSO, Kiro supports that too.

## Open your infrastructure repository

```
File > Open Folder > /path/to/your/infra-repo
```

Or from the terminal:
```bash
kiro /path/to/your/infra-repo
```

Kiro will scan for a `.kiro/` directory at the root. If it finds one it loads steering files and settings automatically. If not, it prompts you to initialize one.

## Initialize Kiro in your repo

If starting fresh, run the Kiro init command:

```
Cmd/Ctrl+Shift+P > "Kiro: Initialize Project"
```

This creates:
```
.kiro/
  steering/        ← Empty directory — you'll populate this
  settings.json    ← MCP servers, agent settings
```

Check this `.kiro/` directory into git. **This is a team artifact**, not a personal preference file. Every engineer who opens the repo in Kiro gets the same context.

## Verify the setup

Open the Kiro chat panel (`Cmd/Ctrl+L`) and ask:

```
What steering files are loaded for this project?
```

Kiro should list the files it found in `.kiro/steering/`. If you just initialized, it'll say none are loaded yet — that's fine, you'll add them in the next steps.

## Connect your AWS credentials

Kiro's MCP servers and some skills need AWS API access. Configure credentials the same way you would for the AWS CLI:

```bash
# Option 1: AWS profile (recommended for local dev)
aws configure --profile my-sandbox

# Option 2: SSO login
aws sso login --profile my-org-sso

# Option 3: Environment variables (CI/CD)
export AWS_ACCESS_KEY_ID=...
export AWS_SECRET_ACCESS_KEY=...
export AWS_DEFAULT_REGION=eu-west-1
```

Kiro inherits credentials from your shell environment and `~/.aws/credentials`. You do not need to paste credentials into Kiro itself.

## Next step

[02 - Understand steering files](./02-steering-files-guide.md)
