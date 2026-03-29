---
name: drift-check
description: Detect and summarise CloudFormation stack drift — differences between deployed resources and their template definitions
---

Check the specified CloudFormation stack for drift and report what has changed outside of CloudFormation.

If the stack name is not in context, ask: "Which stack would you like to check for drift? Provide the stack name and AWS region."

## Steps

1. Use the `cloudformation` MCP tool to initiate a drift detection on the stack:
   - Call `detect_stack_drift` with the stack name
   - Wait for detection to complete (poll `describe_stack_drift_detection_status`)

2. Once complete, call `describe_stack_resource_drifts` to get the full drift report.

3. Analyse the results and produce a report.

## Output format

### Drift Summary

| Status | Count |
|---|---|
| DRIFTED resources | X |
| IN_SYNC resources | X |
| NOT_CHECKED resources | X |
| Detection completed | [timestamp] |

### Drifted Resources

For each drifted resource:

```
Resource: [LogicalResourceId] ([ResourceType])
Physical ID: [id]
Drift status: MODIFIED / DELETED / NOT_FOUND

Property differences:
  Property: [PropertyPath]
    Expected (template): [value]
    Actual (live):       [value]
```

### Risk Assessment

Classify each drifted resource:

| Resource | Drift type | Risk | Recommended action |
|---|---|---|---|
| MySecurityGroup | Security group rule added | HIGH | Who added this? Remove if unauthorized, or update template to codify it |
| MyECSService | DesiredCount changed | LOW | Auto-scaling changed this — consider excluding from drift check |

**Risk levels:**
- **HIGH**: Security group rules added/removed, IAM policy changes, encryption settings changed, public access settings changed
- **MEDIUM**: Configuration changes that could affect behaviour (instance types, environment variables)
- **LOW**: Scaling changes (task counts, ASG sizes) — often expected from auto-scaling

### Recommended actions

For each HIGH/MEDIUM drift, provide one of:
1. **Revert** — the change was unauthorized, roll back with `aws cloudformation drift-detect` + `update-stack`
2. **Codify** — the change was intentional, update the template to match reality
3. **Ignore** — the change is expected (e.g., ASG desired count from auto-scaling)

### How to fix drift

```bash
# Option 1: Revert drifted resources by redeploying the stack
aws cloudformation deploy \
  --stack-name {stack-name} \
  --template-file template.yaml \
  --parameter-overrides ... \
  --capabilities CAPABILITY_IAM

# Option 2: Import the drift into the template (for intentional changes)
# Edit template.yaml to match the actual state, then redeploy
```
