# workflows

What every irmahs-labs repo shares on GitHub. New repos start from [irmahs-labs/template](https://github.com/irmahs-labs/template), which already calls these.

## App CI

`.github/workflows/app-ci.yml` holds the checks every app runs on a pull request, one job per check: **Lint and format**, **Type-check**, **Tests**, **Build** and **Docker build**. A repo's own `ci.yml` calls it, passes only what differs from the defaults, and adds its own jobs next to it:

```yaml
jobs:
  app:
    uses: irmahs-labs/workflows/.github/workflows/app-ci.yml@main
    with:
      node-version: "22"

  smoke:                     # anything only this app needs
    ...

  checks:                    # required by the branch rules
    name: checks
    if: always()
    needs: [app, smoke]
    runs-on: ubuntu-24.04
    steps:
      - if: contains(needs.*.result, 'failure') || contains(needs.*.result, 'cancelled') || contains(needs.*.result, 'skipped')
        run: exit 1
```

| Input | Default | |
|---|---|---|
| `node-version` | `24` | |
| `lint` | `npm run check` | Ultracite |
| `typecheck` | `npm run typecheck` | |
| `test` | `npm test` | |
| `build` | `npm run build` | |
| `compose-check` | `docker compose config --quiet` | runs before the Docker build |
| `docker-image` | `app` | tag for `docker build` |

Set a command to `""` to skip its job, for example `build: ""` when the app's own job builds and smoke-tests it.

Every repo uses `@main`, so a change here reaches all of them on their next run. Check it on one repo's pull request (point its `uses:` at your branch) before merging it here.

## CodeQL

`.github/workflows/codeql.yml` runs [CodeQL](https://codeql.github.com/) code scanning: GitHub's security analysis of the repo's TypeScript and its GitHub Actions workflows. Findings show in the repo's **Security** tab and on pull requests. A repo calls it from its own `.github/workflows/codeql.yml`, on each pull request, each push to `main` and once a week:

```yaml
name: CodeQL

on:
  pull_request:
  push:
    branches: [main]
  schedule:
    - cron: "17 4 * * 1"

jobs:
  codeql:
    uses: irmahs-labs/workflows/.github/workflows/codeql.yml@main
    permissions:
      actions: read
      contents: read
      security-events: write
```

| Input | Default | |
|---|---|---|
| `languages` | `["actions", "javascript-typescript"]` | JSON list; `["actions"]` for a repo with no TypeScript |

Code scanning is free on public repos only: a private repo on the free plan leaves this file out.

## Dependabot

Each repo has a `.github/dependabot.yml` (the template's is the model): npm packages and GitHub Actions, checked once a month. Minor and patch updates come grouped in one pull request per ecosystem; a major version comes on its own, since it can break things. Docker images are left out on purpose: their tags are major versions (`node:24-alpine`, `postgres:17-alpine`), which pick up fixes on every rebuild, and a major image upgrade (above all Postgres) is a deliberate change. For the same reason `@types/node` never moves a major on its own: it follows the Node version in the Dockerfile and `node-version`.
