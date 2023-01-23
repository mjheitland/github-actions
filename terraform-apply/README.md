# terraform-apply

> Action to run terraform apply for a root module based on an existing Terraform plan file.

It runs the following steps:

- terraform apply (using `.artifacts/terraform.plan` that was created from a previous GitHub action `terraform-plan`)
- publish Terraform outputs into `.artifacts` folder

## Usage

```yaml
- uses: mjheitland/github-actions/terraform-apply@v1
  with:
    # Name of the Terraform plan file.
    # Default: .artifacts/terraform.plan
    # Required: No
    plan-file: .artifacts/terraform.plan

    # The relative path in the repository that hosts the terraform code.
    # Default: . (root of the repository)
    # Required: No
    working-directory: '.'
```

## Required permissions

This action requires the following permissions for the default GITHUB_TOKEN.

```yaml
permissions:
  # checkout the repository
  contents: read

  # (optional) allow github actions to assume an iam role
  id-token: write

  # post status of plan
  issues: write

  # post tfsec comments in pull requests
  pull-requests: write

  # allow security events to be published
  security-events: write
```

## Scenarios

### Plan and apply Terraform configuration with one job

This example assumes the Terraform code is stored in the top folder (if not, pass `working-directory` as a parameter).
The workflow is triggered manually.

```yaml
name: apply

on:
  workflow_dispatch:

permissions:
  contents: read
  id-token: write
  issues: write
  pull-requests: write
  security-events: write

# true = cancel any running job and start a new job with the current source code
concurrency:
  group: ${{ github.workflow }}
  cancel-in-progress: false

jobs:
  apply:
    name: apply
    runs-on: ubuntu-latest
    steps:
      - name: checkout
        uses: actions/checkout@v3

      - name: assume-role
        uses: aws-actions/configure-aws-credentials@v1
        with:
          role-to-assume: ${{ secrets.DEPLOY_ROLE }}
          aws-region: ${{ secrets.REGION }}

      - name: grant-private-repo-access
        uses: webfactory/ssh-agent@v0.7.0
        with:
          ssh-private-key: ${{ secrets.SECRET_REPO_DEPLOY_KEY }}

      - name: plan
        uses: mjheitland/github-actions/terraform-plan@v1
        with:
          backend: ${{ secrets.BACKEND }}
          workspace: ${{ secrets.WORKSPACE }}

      - name: apply
        uses: mjheitland/github-actions/terraform-apply@v1
```
