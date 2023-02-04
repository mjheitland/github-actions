# terraform-test

> Represents a GitHub reusable workflow triggered either manually or on pull
  requests to `main` to test infrastructure deployments.
  All subfolders of `examples` are automatically deployed and torn down running `terraform apply` and `terraform destroy`.
  If some of the examples have shared resources that take a while to be set up (e.g. AWS Managed Directory), this code
  can be separated in a `setup` folder.

It runs the following steps:

- test
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
    # go-version:
    #   description: Go version used for running the terratest tests.
    #   default: "1.18"
    #   required: false
    # terraform-version:
    #   description: Terraform version (default is 'latest')
    #   default: latest
    #   required: false
```

## Permissions

```yaml
permissions:
  contents: read
  id-token: write
  issues: write
  pull-requests: write
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

      - name: assume-role
        uses: aws-actions/configure-aws-credentials@v1-node16
        with:
          role-to-assume: ${{ secrets.DEPLOY_ROLE }}
          aws-region: ${{ secrets.REGION }}

      - name: terraform-test
        uses: mjheitland/github-actions/terraform-test@v1
```
