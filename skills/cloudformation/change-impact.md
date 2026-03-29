---
name: change-impact
description: Assess the blast radius and deployment risk of changes to a CloudFormation template
---

Analyse the CloudFormation template changes that are currently staged or shown in the diff.

If a diff is not visible, ask: "Please paste the git diff or describe the changes you're making."

## Change classification

For each changed resource, classify the change:

| Change type | CloudFormation behaviour | Risk |
|---|---|---|
| Property update (no replacement) | In-place update | Low |
| Property update (requires replacement) | Resource deleted and recreated | HIGH |
| Resource addition | New resource created | Low |
| Resource removal | Resource deleted | HIGH if stateful |
| Type change | Resource replaced | HIGH |

Use the CloudFormation documentation to determine whether each property change requires resource replacement. Flag any changes that will cause a replacement with **[REPLACEMENT]**.

## Impact analysis

For each changed resource:

1. **What changes** — old value → new value
2. **Replacement required?** — YES/NO (if YES, what data or state is lost?)
3. **Downtime expected?** — YES/NO/POSSIBLE (with explanation)
4. **Rollback possible?** — YES/NO (DeletionPolicy: Retain changes can't be auto-rolled back)
5. **Dependencies affected** — other resources or stacks that import this resource's exports

## Deployment risk score

| Risk level | Criteria |
|---|---|
| GREEN | Only additions or non-replacement updates, no stateful changes |
| AMBER | Replacement of stateless resources, or multiple changes at once |
| RED | Replacement of stateful resources, removal of resources with exports, changes to IAM roles in use |

## Recommended deployment approach

Based on the risk level, recommend:
- GREEN: Standard pipeline, deploy directly
- AMBER: Deploy to staging first, validate, then prod. Include a rollback plan.
- RED: Manual review required. Consider blue/green approach. Document change window and rollback steps.

## Pre-deployment checklist

Generate a checklist specific to these changes:
- [ ] Stack exports checked — nothing imports the changed outputs
- [ ] Backup taken for any stateful resources being replaced
- [ ] Maintenance window communicated (if downtime expected)
- [ ] Rollback tested in staging
- [ ] Change ticket raised (for prod)
