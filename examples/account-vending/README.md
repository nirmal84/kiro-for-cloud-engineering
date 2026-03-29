# Account Vending Examples

## How to request a new AWS account

1. Copy [account-definition-template.yaml](./account-definition-template.yaml)
2. Rename it: `{org-prefix}-{team}-{environment}.yaml`
3. Replace all placeholder values
4. Submit a PR to this repo — the platform team will review within 1 business day
5. Once merged, the pipeline provisions the account within ~20 minutes

## What gets provisioned automatically

| Component | Time | Who manages |
|---|---|---|
| AWS account creation in Org | 5 min | Pipeline |
| Account alias and contact details | 1 min | Pipeline |
| IAM Identity Center SSO assignments | 2 min | Pipeline |
| Security baseline stack | 5 min | Platform team CFN |
| Network baseline stack (VPC, subnets, TGW) | 8 min | Platform team CFN |
| IAM baseline (permission boundary, audit roles) | 2 min | Platform team CFN |
| Budget alert | 1 min | Platform team CFN |

## Asking Kiro to help with account vending

With the platform steering files loaded in your repo, you can ask Kiro:

```
Create an account definition for a new dev environment for the data engineering team.
They need a VPC in eu-west-1, a budget of $2000/month, and access for the
aws-data-engineering-dev group with DeveloperAccess permission set.
```

Kiro will generate the YAML using the correct format, org-standard CIDR ranges, and required tags — ready to PR.

## Updating an existing account

To change SSO assignments, budget, or other account settings:
1. Edit the existing account definition file
2. Submit a PR
3. Pipeline applies the delta (only changed resources are updated)

## Decommissioning an account

To decommission a sandbox or development account:
1. Submit a PR removing the account definition file
2. Platform team verifies there are no active workloads
3. Pipeline moves account to the `suspended` OU, disables console access
4. After 30-day retention period, account is permanently closed

**You cannot permanently close an account via a PR alone** — this requires a second approval from a platform admin for safety.
