# Shared Node CI

Reusable GitHub Actions workflow for PocketMech (AeroTool) component repos (`mobile-app`, `core`, `sync-service`, `admin-webterminal`, `api`, and others).

Workflow file: [`.github/workflows/ci-node.yml`](../.github/workflows/ci-node.yml)

## Caller snippet

Add this as `.github/workflows/ci.yml` in each component repo:

```yaml
name: CI

on:
  pull_request:
    branches: [develop, main]
  push:
    branches: [develop, main]

jobs:
  ci:
    uses: PocketMech/.github/.github/workflows/ci-node.yml@develop
```

Optional inputs (defaults shown):

```yaml
jobs:
  ci:
    uses: PocketMech/.github/.github/workflows/ci-node.yml@develop
    with:
      node-version: "20"
      working-directory: "."
      package-manager: npm # npm | pnpm | yarn
```

## Behavior

- Runs checkout, Node setup (with dependency caching), lockfile-aware install, then `lint`, `typecheck`, `test`, and `build`.
- Each of those four steps runs only if the matching script exists in `package.json` (via `npm pkg get scripts.<name>`). Missing scripts are skipped with a notice; the job still succeeds.
- If there is no `package.json` in `working-directory`, the workflow succeeds and posts a clear notice that Node CI was skipped.
- Permissions are least-privilege (`contents: read`). Concurrent runs for the same ref cancel superseded ones.
