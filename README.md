# workflows

What every irmahs-labs repo shares on GitHub. New repos start from [irmahs-labs/template](https://github.com/irmahs-labs/template), which already calls these.

## App CI

`.github/workflows/app-ci.yml` holds the checks every app runs on a pull request, one job per check: **Lint and format**, **Type-check**, **Tests**, **Build** and **Docker build**. A repo's own `ci.yml` calls it, passes only what differs from the defaults, and adds its own jobs next to it:

```yaml
jobs:
  app:
    uses: irmahs-labs/workflows/.github/workflows/app-ci.yml@main
    with:
      typecheck: npm run lint

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
