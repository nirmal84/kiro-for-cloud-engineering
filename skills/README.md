# Kiro Skills for Cloud Platform & SRE Teams

Reusable AI workflows for Cloud Engineers, SREs, and DevOps teams — invoked as slash commands or auto-triggered by Kiro when the request matches.

Every skill in this folder follows the [Kiro Agent Skills](https://kiro.dev/docs/skills/) convention — part of the portable [Agent Skills open standard](https://agentskills.io): a folder containing a `SKILL.md` file with YAML frontmatter.

## Starter skill catalogue

| Skill | Slash command | What it does |
|---|---|---|
| [cfn-review](./cfn-review/SKILL.md) | `/cfn-review` | Security, compliance, cost, and reliability review of a CloudFormation template |
| [cfn-explain](./cfn-explain/SKILL.md) | `/cfn-explain` | Plain-English explanation of a template — architecture, cost, access |
| [change-impact](./change-impact/SKILL.md) | `/change-impact` | Blast radius and replacement analysis for a template diff |
| [drift-check](./drift-check/SKILL.md) | `/drift-check` | Detect and classify stack drift using the CloudFormation MCP server |
| [cost-estimate](./cost-estimate/SKILL.md) | `/cost-estimate` | Rough monthly cost estimate for a template or proposed architecture |
| [capacity-report](./capacity-report/SKILL.md) | `/capacity-report` | Monthly capacity, utilisation, and rightsizing report |
| [iam-review](./iam-review/SKILL.md) | `/iam-review` | Least-privilege review of IAM roles, policies, and trust relationships |
| [security-scan](./security-scan/SKILL.md) | `/security-scan` | CIS AWS Foundations, NIST, and OWASP Cloud Top 10 checks |
| [runbook](./runbook/SKILL.md) | `/runbook` | Generate an SRE runbook for a service from its template |
| [incident](./incident/SKILL.md) | `/incident` | Incident status update or post-mortem draft |

## How to load these skills in Kiro

Kiro skills live in one of two locations:

| Location | Scope |
|---|---|
| `.kiro/skills/` (in your repo root) | **Workspace** — available only inside that project |
| `~/.kiro/skills/` (home directory) | **Global** — available in every workspace |

Pick one based on whether the skill is org-wide (global) or repo-specific (workspace). Platform/SRE teams usually want these global; project-specific variants go in the workspace.

### Option A — per-repo (workspace) install

Copies every starter skill into your infrastructure repo's workspace config:

```bash
# From the root of your infrastructure repo
mkdir -p .kiro/skills
cp -R /path/to/kiro-for-cloud-teams/skills/*/ .kiro/skills/
```

The trailing `/*/` is deliberate — it copies each skill folder (e.g. `cfn-review/`) into `.kiro/skills/` without nesting the top-level `skills/` folder or pulling in this `README.md`.

### Option B — user-global install

Makes the skills available in every workspace you open:

```bash
mkdir -p ~/.kiro/skills
cp -R /path/to/kiro-for-cloud-teams/skills/*/ ~/.kiro/skills/
```

### Option C — import via the Kiro UI

In the Kiro IDE:

1. Open the **Agent Steering & Skills** panel (left sidebar)
2. Click **+** → **Import skill**
3. Point to either a local folder or a GitHub repo URL

### Verify they loaded

1. Open any file in Kiro and start a chat
2. Type `/` — you should see `cfn-review`, `cfn-explain`, etc. in the slash-command menu
3. Or just describe what you want ("review this template for security issues") — Kiro's progressive loader matches your request against each skill's `description` and auto-activates the best fit

Only the `name` and `description` are read at startup; the full `SKILL.md` body is loaded on demand. That means you can keep long, detailed skills without slowing Kiro down.

## Anatomy of a skill

Every `SKILL.md` in this folder looks like this:

```markdown
---
name: cfn-review
description: Review a CloudFormation template for security, cost, and compliance issues
---

<the instructions Kiro will follow when this skill is invoked>
```

Rules the spec enforces:

- **`name`** — lowercase letters, digits, and hyphens only. Max 64 chars. **Must match the folder name exactly.**
- **`description`** — max 1024 chars. This is how Kiro decides when to auto-activate the skill. Write it as a clear "when to use" sentence, not marketing copy.

Optional frontmatter keys Kiro accepts: `license`, `compatibility`, `metadata`.

## Authoring a new skill

1. Create a folder with the skill name: `mkdir skills/my-skill`
2. Create `skills/my-skill/SKILL.md` with frontmatter (`name` matching the folder) and the prompt body
3. Drop the folder into `.kiro/skills/` (workspace) or `~/.kiro/skills/` (global), or import via the Agent Steering & Skills panel
4. Restart Kiro or reload the panel — the new skill appears in the `/` menu

### Optional skill assets

A skill can be more than just a prompt. The spec allows these sibling directories inside the skill folder:

```
my-skill/
├── SKILL.md           # required — the prompt + frontmatter
├── scripts/           # executable helpers the skill can invoke
├── references/        # additional docs Kiro can consult
└── assets/            # templates, snippets, images
```

## Writing tips for cloud/SRE skills

**Be output-format specific.** "Output a markdown table with columns: Resource | Issue | Severity | Fix" beats "list the issues." See [cfn-review](./cfn-review/SKILL.md) for a template.

**Encode your org's standards in the skill.** A good skill references your specific tags, permission boundary ARNs, SSM parameter paths, and account conventions — not generic AWS examples. This is the multiplier over plain ChatGPT.

**Chain with MCP tools.** Skills can instruct Kiro to call MCP servers mid-execution. The [drift-check](./drift-check/SKILL.md) and [capacity-report](./capacity-report/SKILL.md) skills show how to combine the `cloudformation` and `cost-analysis` MCP servers with a skill prompt.

**One skill, one job.** If your skill does "review AND fix AND explain," split it. Small focused skills compose better and auto-trigger more reliably.

**Keep `description` sharp.** Kiro matches the user's request against descriptions to pick a skill. Vague descriptions mean the skill never fires; specific ones fire at the right moment.

## Further reading

- [Kiro Agent Skills documentation](https://kiro.dev/docs/skills/)
- [Kiro CLI Skills documentation](https://kiro.dev/docs/cli/skills/)
- [Agent Skills open standard](https://agentskills.io)
- [Getting started guide: Step 6 — Creating skills](../getting-started/06-skills-guide.md)
