# Step 2: Understanding Steering Files

## What are steering files?

Steering files are markdown documents in `.kiro/steering/` that tell Kiro how your organisation does things. They're the difference between Kiro generating generic AWS examples and Kiro generating templates that pass your internal review on the first try.

Think of them as a persistent system prompt that travels with your repo.

## How Kiro loads them

```
.kiro/steering/
  aws-standards.md          ← Always active (no frontmatter = global)
  cfn-templates.md          ← Always active
  prod-access.md            ← Always active
```

Any `.md` file in `.kiro/steering/` is loaded automatically when Kiro opens the project. There's no registration step.

## Conditional steering with `includeBy`

You can scope steering to specific file patterns so it only activates when relevant:

```markdown
---
includeBy:
  - "**/*.yaml"
  - "**/*.yml"
  - "**/cloudformation/**"
---

# CloudFormation Standards

Only apply these rules when working with YAML files or CFN directories.
...
```

This keeps Kiro's context focused. A 50-file steering library won't slow things down because irrelevant files aren't loaded.

## What makes a good steering file?

**Be specific about your org's choices, not AWS best practices generally.** Kiro already knows AWS best practices. It doesn't know that your org:

- Uses `eu-west-1` as primary and `eu-west-2` as DR
- Tags everything with `CostCentre`, `Team`, and `Environment`
- Deploys via GitHub Actions with an OIDC role, not access keys
- Requires a `DeletionPolicy: Retain` on all stateful resources in prod
- Uses a specific VPC CIDR scheme across accounts

**Structure that works well:**

```markdown
# [Domain] Standards

## Non-negotiables
- Rule 1 (no exceptions)
- Rule 2 (no exceptions)

## Conventions
- Preferred pattern A over B because [reason]
- Name things like X-{env}-{region}-{purpose}

## Examples
[Short inline examples of the right pattern]

## Anti-patterns to avoid
- Don't do X because it causes Y
```

## Anatomy of a steering file

```markdown
---
includeBy:           # Optional: file globs that trigger this file
  - "**/cfn/**"
---

# Title

Short description of what this steering covers.

## Context
Why these rules exist (brief — Kiro doesn't need the full history).

## Rules
Numbered or bulleted. Concrete. Opinionated.

## Examples
Inline code blocks work well here.
```

## The three tiers of steering

For cloud platform teams we recommend three tiers:

| Tier | Location | Maintained by | Examples |
|---|---|---|---|
| **Org-wide** | Shared as a git submodule or copied to every repo | Platform team | Tagging policy, account IDs, region list, IAM boundary ARNs |
| **Repo-level** | `.kiro/steering/` in the repo | Repo owners | What this repo deploys, its deployment pipeline, its specific patterns |
| **Personal** | `~/.kiro/steering/` | Individual | Your own shortcuts, preferences (not checked in) |

## Next step

[03 - Set up CloudFormation steering](./03-cloudformation-steering.md)
