# run-neophile

This is a [composite GitHub Action](https://docs.github.com/en/actions/creating-actions/creating-a-composite-action) for running [neophile](https://neophile.lsst.io/) to analyze or update dependencies.

The action:

1. Sets up Python
2. Installs or updates neophile
3. Runs neophile with appropriate options

Three modes of operation are supported:

- **check** runs `neophile check`, which fails if dependencies are out of date
- **update** runs `neophile update` to update dependencies in place
- **pr** runs `neophile update --pr`, which creates a pull request to update dependencies

By default, all types of dependencies known to neophile are checked or updated.
This can be limited to a space-separated list of dependency types using the `types` setting.

## Example usage

To check whether Python dependencies are up-to-date:

```yaml
name: Python CI

"on":
  pull_request: {}

jobs:
  dependencies:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v3

      - name: Run neophile
        uses: lsst-sqre/run-neophile@v1
        with:
          python-version: "3.11"
          mode: check
          types: python
```

To update all dependencies, such as in preparation for running the test suite to see if tests would fail once dependencies are updated:

```yaml
name: Periodic CI

"on":
  schedule:
    - cron: "0 12 * * 1"
  workflow_dispatch: {}

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v3

      - name: Run neophile
        uses: lsst-sqre/run-neophile@v1
        with:
          python-version: "3.11"
          mode: update
```

To create a PR to update pre-commit dependencies (if necessary):

```yaml
name: Dependency Update
"on":
  schedule:
    - cron: "0 12 * * 1"
  workflow_dispatch: {}

jobs:
  update:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v3

      - name: Run neophile
        uses: lsst-sqre/run-neophile@v1
        with:
          python-version: "3.11"
          mode: pr
          types: pre-commit
          app-id: ${{ secrets.NEOPHILE_APP_ID }}
          app-secret: ${{ secrets.NEOPHILE_PRIVATE_KEY }}
```

## Inputs

- `python-version` (string, required): The Python version.
- `mode` (string, required): The mode of operation chosen from `check`, `update`, or `pr`
- `types` (string, optional): Limit neophile's operation to these space-separated dependency types, chosen from `pre-commit` and `python` (default: all supported types)
- `working-directory` (string, optional): Directory to run in (default: repository root)
- `app-id` (string, optional, required if mode is `pr`): GitHub App id to run neophile as, needed for creating PRs
- `app-secret` (string, optional, required if mode is `pr`): GitHub App private key, needed for creating PRs

## Outputs

No outputs.

## Developer guide

This repository provides a **composite** GitHub Action, a type of action that packages multiple regular actions into a single step.
We do this to make the GitHub Actions workflows of all our software projects more consistent and easier to maintain.
[You can learn more about composite actions in the GitHub documentation.](https://docs.github.com/en/actions/creating-actions/creating-a-composite-action)

Create new releases using the GitHub Releases UI and assign a tag with a [semantic version](https://semver.org), including a `v` prefix.
Choose the semantic version based on compatibility for users of this workflow.
If backwards compatibility is broken, bump the major version.

When a release is made, a new major version tag (i.e. `v1`, `v2`) is also made or moved using [nowactions/update-majorver](https://github.com/marketplace/actions/update-major-version).
We generally expect that most users will track these major version tags.
