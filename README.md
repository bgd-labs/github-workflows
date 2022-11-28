# BGD <> Github workflows

This repository contains a collection of workflows that are used across BGD repositories.
The workflows can easily be used outside of BGD as well, as long as secrets are properly setup.

## Foundry test

`foundry-test` workflow:

- installs foundry
- runs `forge build --sizes`
- runs `forge test -vvv`
- reports results in an automatically updated comment on the pr
- caches fork snapshots for recurrent runs

You can use the workflow via:

```yml
jobs:
  test:
    uses: bgd-labs/github-workflows/.github/workflows/foundry-test.yml@main
```

Per default the workflow will assume your `.env.example` contains variables that are sufficient to run your tests. For advanced usage you can specify a custom `testCommand` and custom `RPCs`.

```yml
jobs:
  test:
    uses: bgd-labs/github-workflows/.github/workflows/foundry-test.yml@main
    with:
      testCommand: make test
    # to inherit all secrets
    secrets: inherit
    # to inherit specific secrets
    secrets:
      RPC_MAINNET: ${{ secrets.RPC_MAINNET }}
```

## Draft release

`draft-release` workflow:

- uses `standard-version` to generate a changelog based on [conventional commit messages](https://www.conventionalcommits.org/en/v1.0.0/)
- creates a pr with the changes and a specific commit message

```yml
# this action is supposed to be used via workflow dispatch
on:
  workflow_dispatch:

jobs:
  draft-release:
    uses: bgd-labs/github-workflows/.github/workflows/draft-release.yml@main
```

## Release / Release-node

`release` and `release-node` actions are intended to be used with `draft-release`.
Instead of relying on manual dispatch this action is intended to be used on default branch merges only.

The workflows will listen for the commit message created by `draft-release` and then perform their release tasks.

`release` workflow:

- creates a github tag

`release-node` workflow:

- creates a node release
- the repository **MUST** use `yarn` as it's package manager
- the repository **MUST** specify a `ci:publish` step which facilitates [building and publishing](https://github.com/bgd-labs/aave-address-book/blob/main/package.json#L17)

```yml
on:
  push:
    branches:
      - main

jobs:
  release:
    uses: bgd-labs/github-workflows/.github/workflows/release.yml@main
  release-node:
    uses: bgd-labs/github-workflows/.github/workflows/release-node.yml@main
    secrets:
      NODE_AUTH_TOKEN: ${{ secrets.NODE_AUTH_TOKEN }}
```
