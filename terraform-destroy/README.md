# terraform-destroy

> Represents a GitHub reusable workflow for destroying infrastructure resources with Terraform.

It runs the following steps:

- terraform init (using `workspace` and `backend`)
- terraform destroy -auto-approve
- publish Terraform messages into `.artifacts` folder

## Usage

```yaml
    - name: destroy
      id: destroy
      shell: bash
      working-directory: ${{ inputs.working-directory }}
      run: |
        terraform destroy -auto-approve
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

### Destroy all previously deployed resources in a given workspace using its backend file

This example assumes the Terraform code is stored in the top folder (if not, pass `working-directory` as a parameter).
The workflow is triggered manually.

```yaml
name: destroy

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
  destroy:
    name: destroy
    runs-on: ubuntu-latest
    steps:
      - name: checkout
        uses: actions/checkout@v3

      - name: assume-role
        uses: aws-actions/configure-aws-credentials@v1-node16
        with:
          role-to-assume: ${{ secrets.DEPLOY_ROLE }}
          aws-region: ${{ secrets.REGION }}

      - name: destroy
        uses: mjheitland/github-actions/terraform-destroy@v1
        with:
          backend: ${{ secrets.BACKEND }}
          workspace: ${{ secrets.WORKSPACE }}
```
