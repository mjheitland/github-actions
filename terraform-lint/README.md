# terraform-lint

> Represents a GitHub reusable workflow usually triggered either manually or on pull
  requests to `main` to run [TFLINT](https://github.com/terraform-linters/tflint).

`TFLint` is a framework and each feature is provided by plugins, the key features are as follows:

- Find possible errors (like invalid instance types) for Major Cloud providers (AWS/Azure/GCP).
- Warn about deprecated syntax, unused declarations.
- Enforce best practices, naming conventions.

This GitHub action runs the following steps:

- TFLint
- publish TFLint results into `.artifacts` folder

## Usage

```yaml
  uses: GDP3-AWS/github-actions/terraform-lint@v1
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

```yaml
name: lint

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
  lint:
    name: lint
    runs-on: [aws]
    steps:
      - name: checkout
        uses: actions/checkout@v3

      - name: terraform-lint
        uses: GDP3-AWS/github-actions/terraform-lint@v1
```
