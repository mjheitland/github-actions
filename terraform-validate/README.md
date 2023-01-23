# terraform-validate

> This action is usually run either manually or on a Pull Request to the `main` branch.
  It runs various static code analyzers (`terraform validate`, `tflint`,
  `checkov`).
  It also checks that `README.md` was auto-generated running `terraform-docs`
  and that the configuration code has been properly formatted with `terraform fmt`.

It runs the following steps:

- terraform-docs (fail on diff)
- terraform fmt -recursive -check (fail on diff)
- terraform init -upgrade
- terraform validate
- tflint
- checkov
- publish results into `.artifacts` folder

## Usage

```yaml
  uses: mjheitland/github-actions/terraform-validate@v1
  with:
    # github-token:
    #   description: |
    #     Optional GitHub Token when elevated permissions are needed that cannot be
    #     provided by the default token
    #   required: false
    #   default: ${{ github.token }}
    # terraform-version:
    #   description: Terraform version (default is 'latest')
    #   default: latest
    #   required: false
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

### Pull Request for a Terraform Module

Trigger on a pull request to main or when run manually.

**NOTE**: Depending on your tests, you might have to assume a role with sufficient permissions before running this workflow.

```yaml
name: validate-and-test-on-pr

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
  validate:
    name: validate
    runs-on: ubuntu-latest
    steps:
      - name: checkout
        uses: actions/checkout@v3

      - name: terraform-validate
        uses: mjheitland/github-actions/terraform-validate@v1
```
