# Step 6: Creating Kiro Skills for Cloud Operations

## What are skills?

Skills are reusable, shareable AI workflows that Kiro can invoke as slash commands (`/cfn-review`) or auto-activate when your request matches the skill's description. Instead of retyping a long prompt every time you review a template, you write it once as a skill and call it by name — or let Kiro pick it for you.

Kiro skills follow the [open Agent Skills standard](https://agentskills.io) — a portable convention for defining AI workflows. That means skills written for this repo work unchanged in any IDE or agent that implements the same standard.

## Where skills live

Kiro looks for skills in two places:

| Location | Scope | When to use |
|---|---|---|
| `.kiro/skills/` | Workspace — only this repo | Repo-specific skills (e.g. tied to a custom service) |
| `~/.kiro/skills/` | Global — every workspace | Org-wide skills that every engineer benefits from |

For a cloud platform team, most starter skills (`cfn-review`, `security-scan`, `runbook`) belong in `~/.kiro/skills/` so they follow engineers across repos. Put repo-specific ones in `.kiro/skills/`.

## Anatomy of a skill

A skill is a **folder** containing a `SKILL.md` file. The folder name must match the `name` field in the frontmatter.

```
.kiro/skills/
└── cfn-review/
    └── SKILL.md
```

A minimal `SKILL.md`:

```markdown
---
name: cfn-review
description: Review a CloudFormation template for security, cost, and compliance issues
---

Review the selected CloudFormation template and check for:

1. **Security issues**
   - Over-permissive IAM (wildcards on Action or Resource)
   - Unencrypted storage (S3, EBS, RDS, DynamoDB)
   - Public-facing resources without explicit justification

2. **Cost risks**
   - Resources with no deletion policy in non-prod
   - Over-provisioned instance types
   - Multi-AZ where single-AZ would suffice for non-prod

3. **Compliance**
   - Missing required tags: Environment, Team, CostCentre, ManagedBy
   - Missing permission boundaries on IAM roles

Output a markdown table: | Resource | Issue | Severity | Recommendation |
```

Frontmatter rules:

- `name` — lowercase letters, digits, hyphens only. Max 64 chars. **Must match the folder name.**
- `description` — max 1024 chars. This is what Kiro matches your request against when auto-activating a skill, so be specific about *when* to use it.

Optional assets the spec allows alongside `SKILL.md`:

```
cfn-review/
├── SKILL.md        # required
├── scripts/        # executable helpers
├── references/     # supporting docs
└── assets/         # templates, snippets
```

## Invoking a skill

**Manual (slash command):** type `/` in the Kiro chat and pick from the menu.

```
/cfn-review
```

**With context:** select a file first, then invoke — the skill runs against the selection.

**Automatic:** just describe what you want. Kiro reads every skill's `description` and activates the best match.

```
"Review this template for security and compliance issues"
  → Kiro auto-picks /cfn-review
```

Kiro loads only the `name` and `description` at startup, so you can keep long detailed skills without startup cost — the full body loads only when the skill is invoked.

## Starter skills in this repo

See the [skills catalogue](../skills/README.md) for the full list with links. Summary:

| Skill | Command | What it does |
|---|---|---|
| [cfn-review](../skills/cfn-review/SKILL.md) | `/cfn-review` | Security, cost, compliance review |
| [cfn-explain](../skills/cfn-explain/SKILL.md) | `/cfn-explain` | Plain-English explanation |
| [cost-estimate](../skills/cost-estimate/SKILL.md) | `/cost-estimate` | Rough monthly cost estimate |
| [security-scan](../skills/security-scan/SKILL.md) | `/security-scan` | CIS/OWASP check on IAM and network config |
| [runbook](../skills/runbook/SKILL.md) | `/runbook` | Generate an SRE runbook |
| [incident](../skills/incident/SKILL.md) | `/incident` | Incident status or post-mortem |
| [drift-check](../skills/drift-check/SKILL.md) | `/drift-check` | Stack drift detection |
| [change-impact](../skills/change-impact/SKILL.md) | `/change-impact` | Blast radius analysis |
| [iam-review](../skills/iam-review/SKILL.md) | `/iam-review` | Least-privilege IAM review |
| [capacity-report](../skills/capacity-report/SKILL.md) | `/capacity-report` | Utilisation and rightsizing report |

## Install the starter skills

**Per-repo (workspace):**

```bash
mkdir -p .kiro/skills
cp -R /path/to/kiro-for-cloud-teams/skills/*/ .kiro/skills/
```

**Globally (every workspace):**

```bash
mkdir -p ~/.kiro/skills
cp -R /path/to/kiro-for-cloud-teams/skills/*/ ~/.kiro/skills/
```

The `/*/` glob copies each skill folder directly — without it you'd nest `skills/skills/...` and pick up the catalogue README too.

**Or import in the UI:** open the Kiro **Agent Steering & Skills** panel → **+** → import from local folder or GitHub URL.

## Verify the install

1. Open any file in Kiro and start a chat
2. Type `/` — the slash menu should list `cfn-review`, `cfn-explain`, etc.
3. Or describe a task ("review this template for security issues") and watch Kiro auto-activate the matching skill

If a skill doesn't appear, check:

- Folder name matches the `name:` field exactly (lowercase + hyphens only)
- `SKILL.md` (exact filename, case sensitive) exists inside the folder
- Frontmatter `name` and `description` are both present and within length limits
- You restarted the Kiro chat session after copying (or reloaded the skills panel)

## Tips for writing effective skills

**Be output-format specific.** "Output a markdown table with columns: Resource | Issue | Severity | Fix" beats "list the issues."

**Encode your org's context.** A good skill references your specific tag keys, permission boundary ARNs, SSM parameter paths, and account conventions — not generic AWS examples. This is what turns a prompt into a force multiplier.

**Chain with MCP tools.** Skills can instruct Kiro to call MCP servers mid-execution:

```markdown
---
name: stack-health
description: Summarise deployment status and cost for a named CloudFormation stack
---
Use the cloudformation MCP tool to describe the stack named {stack_name}.
Then use the cost-analysis MCP tool to get the last 30 days cost for resources
tagged Stack={stack_name}.
Report: deployment date, resource count, monthly cost, any failed resources.
```

**Keep each skill focused.** One skill, one job. If yours does review AND fix AND explain, split it — small focused skills compose better and auto-trigger more reliably.

**Write the description as a trigger.** Kiro matches the user's request against the description field. Write it as "when to use this" rather than "what this does." Good: *"Review a CloudFormation template for security, cost, and compliance issues."* Poor: *"The cfn-review skill."*
