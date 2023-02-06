# terraform-plan

> A GitHub reusable workflow to run `terraform plan`.

It runs the following steps:

- terraform init -upgrade (update TF modules while considering terraform workspace and backend specification)
- terraform plan (optional: includes run of `terraform-validate` to do a static code analysis)
- publish results into `.artifacts` folder

> **NOTE**: Before you run the `terraform-plan` action, you have to check out your Terraform configuration code and
> assume a role with sufficient permissions to run `terraform plan`.

## Usage

```yaml
- uses: mjheitland/github-actions/terraform-plan@v1
  with:
    # Path to the Terraform backend file relative to the working directory.
    # Required: No
    backend: ''

    # Optional GitHub Token when elevated permissions are needed
    # that cannot be provided by the default token
    # Required: No
    github-token: ${{ github.token }}

    # Do you want to run `terraform plan` with `-destroy` option?
    # Required: No
    run-terraform-plan-with-destroy-option: 'false'

    # Before running `terraform plan`, do a static code analysis by running `terraform-validate`
    # Required: No
    run-terraform-validate: 'true'

    # The Terraform version to install and use in the pipeline. Make sure to use the same version
    # for both plan and apply actions.
    # Required: No
    terraform-version: 'latest'

    # The relative path in the repository that hosts the terraform code.
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

### Validate and plan infrastructure deployment using Terraform workspace and backend

```yaml
name: validate-and-plan

on:
  workflow_dispatch:

  pull_request_target:
    branches:
      - main

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

      - name: terraform-plan
        uses: mjheitland/github-actions/terraform-plan@v1
        with:
          backend: ${{ secrets.BACKEND }}
          workspace: ${{ secrets.WORKSPACE }}
```
