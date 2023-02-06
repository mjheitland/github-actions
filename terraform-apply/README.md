# terraform-apply

> Action to run terraform apply for a root module based on an existing Terraform plan file.

It runs the following steps:

- terraform apply (using `.artifacts/terraform.plan` that was created from a previous GitHub action `terraform-plan`)
- publish Terraform outputs into `.artifacts` folder

## Usage

```yaml
- uses: mjheitland/github-actions/terraform-apply@v1
  with:
    # Path to the Terraform backend file relative to the working directory.
    # Required: No
    backend: ''

    # Optional GitHub Token when elevated permissions are needed
    # that cannot be provided by the default token
    # Required: No
    github-token: ${{ github.token }}

    # Name of the Terraform plan file.
    # Default: .artifacts/terraform.plan
    # Required: No
    plan-file: .artifacts/terraform.plan

    # Do you want to run `terraform apply` with `-destroy` option?
    # Required: No
    run-terraform-apply-with-destroy-option: 'false'

    # The relative path in the repository that hosts the terraform code.
    # Default: . (root of the repository)
    # Required: No
    working-directory: '.'

    # The Terraform workspace name
    # Required: No
    workspace: 'default'
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
  id-token: write
  contents: read
  issues: write
  pull-requests: write
  security-events: write

# true = cancel any running job and start a new job with the current source code
concurrency:
  group: ${{ github.workflow }}
  cancel-in-progress: false

jobs:

  validate-and-plan:
    name: validate-and-plan
    runs-on: ubuntu-latest
    steps:
      - name: checkout
        uses: actions/checkout@v3

      - name: assume-role
        uses: aws-actions/configure-aws-credentials@v1-node16
        with:
          role-to-assume: ${{ secrets.DEPLOY_ROLE }}
          aws-region: ${{ secrets.REGION }}

      - name: validate-and-plan
        uses: mjheitland/github-actions/terraform-plan@v1
        with:
          backend: ${{ secrets.BACKEND }}
          workspace: ${{ secrets.WORKSPACE }}

  apply:
    name: apply
    environment: dev
    runs-on: ubuntu-latest
    needs: [validate-and-plan]
    steps:
      - name: prepare
        id: prepare
        shell: bash
        run: |
          # create artifacts directory
          # (won't be changed if it already exists from a previous run of `terraform-plan`)
          mkdir -p .artifacts
          ls -al .artifacts

      - name: checkout
        uses: actions/checkout@v3

      - name: assume-role
        uses: aws-actions/configure-aws-credentials@v1-node16
        with:
          role-to-assume: ${{ secrets.DEPLOY_ROLE }}
          aws-region: ${{ secrets.REGION }}

      - name: download-plan
        uses: actions/download-artifact@v3
        with:
          name: plan-artifacts
          path: .artifacts

      - name: show
        shell: bash
        working-directory: ${{ inputs.working-directory }}
        run: |
          ls -al .artifacts
        continue-on-error: true

      - name: apply
        uses: mjheitland/github-actions/terraform-apply@v1
        with:
          backend: ${{ secrets.BACKEND }}
          workspace: ${{ secrets.WORKSPACE }}
```
