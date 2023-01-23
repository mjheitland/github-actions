# terraform-release

Generates a PR to create a new release with three tags all pointing to the last commit
based on [semantic versioning](https://semver.org/)
and [conventional commits](https://www.conventionalcommits.org/en/v1.0.0/#specification):

1. `major`
2. `major.minor`
3. `major.minor.patch`

Example:
Previous version was `v1.2.2`.
After a commit with a message "fix(example): fix url" and releasing the module, a new PR will be puslished.
Once this PR got approved, a GitHub action starts automatically:
it creates a release and attaches three new tags `v1`, `v1.2` and `v1.2.3` pointing to the merge commit.

The key words (all lower case!) in the commit messages that should be used are

- docs: does not change version numbers
- fix: increase patch number
- feat: increase minor number
- fix! or feat!: increase major number

This GitHub action runs the following steps:

- create a PR for a new release with the next version tag (e.g. `v1.2.3`) leveraging [google-github-actions](https://github.com/google-github-actions/release-please-action)
- checkout
- create two additional tags (e.g. `v1` and `v1.2` pointing to the same commit as `v1.2.3`)

## Usage

```yaml
- uses: mjheitland/github-actions/terraform-release@v1
```

## Permissions

```yaml
permissions:
  # checkout the repository (checkout is not handled by this action!)
  contents: read

  # post tfsec comments in pull requests
  packages: write

  # allow security events to be published
  pull-requests: write
```

## Scenarios

### Increment semantic version and add two additional major/major-minor tags pointing to the last release

```yaml
name: release-main

on:
  workflow_dispatch:

  push:
    branches:
      - main

permissions:
  contents: write
  packages: write
  pull-requests: write

jobs:
  release:
    runs-on: ubuntu-latest
    if: "! contains(github.event.head_commit.message, '[ci skip]') && ! contains(github.event.head_commit.message, 'docs:')"
    steps:
      - name: release-module
        uses: mjheitland/github-actions/terraform-release@v1
```
