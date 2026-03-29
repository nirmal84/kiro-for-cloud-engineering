---
includeBy:
  - "**/.github/workflows/**"
  - "**/pipeline/**"
  - "**/deploy/**"
  - "**/ci/**"
---

# Deployment Pipeline Standards

## Never use long-lived access keys in CI/CD

All GitHub Actions deployments use OIDC federation. This is non-negotiable.

```yaml
# CORRECT: OIDC federation
- name: Configure AWS credentials
  uses: aws-actions/configure-aws-credentials@v4
  with:
    role-to-assume: arn:aws:iam::${{ vars.AWS_ACCOUNT_ID }}:role/GitHubActions-Deploy-${{ env.ENVIRONMENT }}
    aws-region: eu-west-1
    role-session-name: GitHubActions-${{ github.run_id }}

# WRONG: never do this
- name: Configure AWS credentials
  uses: aws-actions/configure-aws-credentials@v4
  with:
    aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
    aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

## Standard deployment workflow structure

Every CloudFormation deployment pipeline must follow this sequence:

```yaml
name: Deploy

on:
  push:
    branches: [main]
  workflow_dispatch:
    inputs:
      environment:
        type: choice
        options: [staging, prod]

jobs:
  validate:
    name: Validate templates
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Run cfn-lint
        run: |
          pip install cfn-lint
          cfn-lint templates/**/*.yaml
      - name: Run cfn-nag
        run: |
          gem install cfn-nag
          cfn_nag_scan --input-path templates/
      - name: Run checkov
        uses: bridgecrewio/checkov-action@master
        with:
          directory: templates/
          framework: cloudformation

  deploy-staging:
    name: Deploy to staging
    needs: validate
    runs-on: ubuntu-latest
    environment: staging
    steps:
      - uses: actions/checkout@v4
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::${{ vars.STAGING_ACCOUNT_ID }}:role/GitHubActions-Deploy-staging
          aws-region: eu-west-1
      - name: Deploy stack
        run: |
          aws cloudformation deploy \
            --template-file templates/main.yaml \
            --stack-name ${{ env.SERVICE }}-staging \
            --parameter-overrides Environment=staging Team=${{ env.TEAM }} \
            --capabilities CAPABILITY_IAM CAPABILITY_NAMED_IAM \
            --tags Environment=staging Team=${{ env.TEAM }} ManagedBy=cloudformation
      - name: Run smoke tests
        run: ./scripts/smoke-test.sh staging

  approve-prod:
    name: Approve production deployment
    needs: deploy-staging
    runs-on: ubuntu-latest
    environment: prod-approval   # GitHub environment with required reviewers
    steps:
      - run: echo "Approved for production"

  deploy-prod:
    name: Deploy to production
    needs: approve-prod
    runs-on: ubuntu-latest
    environment: prod
    steps:
      - uses: actions/checkout@v4
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::${{ vars.PROD_ACCOUNT_ID }}:role/GitHubActions-Deploy-prod
          aws-region: eu-west-1
      - name: Deploy stack
        run: |
          aws cloudformation deploy \
            --template-file templates/main.yaml \
            --stack-name ${{ env.SERVICE }}-prod \
            --parameter-overrides Environment=prod Team=${{ env.TEAM }} \
            --capabilities CAPABILITY_IAM CAPABILITY_NAMED_IAM \
            --tags Environment=prod Team=${{ env.TEAM }} ManagedBy=cloudformation
      - name: Run smoke tests
        run: ./scripts/smoke-test.sh prod
```

## GitHub environment protection rules

Configure these in GitHub repository settings:

**Staging environment:**
- No protection rules (auto-deploys from main)
- Required reviewers: none

**Prod-approval environment:**
- Required reviewers: 2 (from CODEOWNERS)
- Wait timer: 0 minutes
- Deployment branches: main only

**Prod environment:**
- Required reviewers: none (approval was the previous gate)
- Deployment branches: main only

## Required pre-deployment checks

All pipelines must run before deploying to any environment:
1. `cfn-lint` — CloudFormation template validation
2. `cfn-nag` — Security rule violations
3. `checkov` — Broader compliance scan

No deployment proceeds if any of these checks fail.

## Rollback strategy

```yaml
# CloudFormation handles rollback automatically on failure
# For manual rollback:
- name: Rollback on failure
  if: failure()
  run: |
    aws cloudformation continue-update-rollback \
      --stack-name ${{ env.STACK_NAME }}
    # Or for complete rollback to previous successful state:
    # aws cloudformation rollback-complete-stack is not a command
    # Use previous changeset or redeploy the previous git tag
```

## Secrets in pipelines

Never put secrets in workflow files. Use:
- GitHub Actions secrets for sensitive values
- `vars.` (GitHub variables) for non-sensitive account IDs, region, service names
- AWS Secrets Manager / SSM Parameter Store for application runtime secrets
