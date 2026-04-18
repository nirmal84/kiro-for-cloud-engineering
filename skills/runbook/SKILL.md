---
name: runbook
description: Generate an SRE runbook for a service based on its CloudFormation template and architecture
---

Generate a production runbook for the service described in the currently open template or file.

If a CloudFormation template is open, use it to understand the architecture. If no template is open, ask: "Which service should I generate a runbook for? Please describe its components or open its CloudFormation template."

Use the `cloudformation` MCP tool if available to check the deployed stack status.

## Runbook structure to generate

---

# [Service Name] Runbook

**Service:** [name]
**Team:** [team]
**On-call rotation:** [link or team name]
**Escalation contact:** [who to call if you can't resolve in 30 minutes]
**SLO:** [availability target, e.g. 99.9%]
**Tier:** [P0/P1/P2/P3]

---

## Service Overview

[2-3 sentences: what this service does, who uses it, what breaks if it goes down]

## Architecture

[Text diagram of the key components and how they connect]

**Key resources:**
- Load balancer: [ARN or name pattern]
- ECS service: [name pattern]
- Database: [name pattern]
- Key external dependencies: [list]

## Dashboards and monitoring

- CloudWatch dashboard: [link pattern]
- Key metrics: [list the 3-5 most important metrics]
- Log groups: [list]

---

## Alert playbooks

### [Alert name 1]

**What it means:** [plain English]
**Severity:** P0/P1/P2
**First response (first 5 minutes):**
1. [Step 1]
2. [Step 2]
**Escalate if:** [condition]
**Common causes:** [list]
**Fix:** [steps]

### [Alert name 2]

[Repeat for each likely alert based on the architecture]

---

## Common operational tasks

### Deploy a new version
```bash
# Commands to trigger a deployment
```

### Roll back a deployment
```bash
# Exact commands to roll back — CloudFormation changeset rollback or ECS force-new-deployment
```

### Scale up/down
```bash
# Commands to adjust capacity
```

### Check health
```bash
# Commands to verify the service is healthy
```

### Access the database (break-glass)
```bash
# How to connect via SSM Session Manager + port forwarding
# NEVER via direct SSH or public RDS access
```

---

## Incident response

1. **Acknowledge** alert in PagerDuty
2. **Join** incident channel: `#inc-[service]-[YYYY-MM-DD]`
3. **Announce** in `#incidents`: "Investigating [alert name] on [service]-[env]"
4. **Diagnose** — follow the relevant alert playbook above
5. **Mitigate** — restore service first
6. **Resolve** and update incident channel
7. **Post-mortem** — create post-mortem doc within 48 hours (P0) or 1 week (P1)

## Post-mortem template link

[Link to your post-mortem template]

---

## Configuration and secrets

| Config item | Location | How to update |
|---|---|---|
| App config | SSM Parameter Store: /[env]/[service]/ | Update SSM, then redeploy |
| Secrets | Secrets Manager: [env]/[service]/ | Rotate via console or CLI |
| Feature flags | [Location] | [How] |

---

*Last updated: [date] by [team]*
*Template source: Kiro /runbook skill*

---

After generating the runbook, note any gaps where the architecture doesn't have runbook entries (e.g., no alert names defined yet, no dashboard configured). List these as **TODO items** at the end.
