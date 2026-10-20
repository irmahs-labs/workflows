# CLAUDE.md

## Git branches

Always pull before making changes: run `git fetch origin` and start from the latest `origin/main` before the first edit.

- On a branch whose PR is open, bring it up to date with `origin/main` first
- On `main`, or on a branch whose PR is already merged, create a new branch from `origin/main` instead

Always name the branch after the feature or fix it contains, never a random or generated name.

- Format: `<type>/<feature-name>` in lowercase kebab-case
- Types: `feat`, `fix`, `chore`, `docs`, `refactor`
- Examples: `feat/e2e-job`, `fix/node-version-input`, `docs/claude-md`
- If the session assigns a generic branch (for example `claude/<random-words>`), create a feature-named branch instead and push there

## Pull requests

- Never push to `main`. Every change goes in through a pull request.
- Open the PR and stop there: never merge it, not even when CI is green. Irma reviews and clicks merge.
- Deploys run on merge to `main`. Re-run a deploy only when Irma asks.

## Checks before pushing

```sh
actionlint   # https://github.com/rhysd/actionlint; the checks job runs it on every pull request
```

## What this is

The CI workflows every irmahs-labs repo calls: `.github/workflows/app-ci.yml` (Lint and format, Type-check, Tests, Build, Docker build) and `.github/workflows/codeql.yml` (code scanning). `README.md` lists their inputs and shows how a repo calls them.

- Every repo calls it at `@main`, so a merge here changes every repo's next CI run at once. Before merging a change, point one repo's `uses:` at the branch in a draft pull request there and see it pass.
- Adding an input with a default is safe. Renaming or removing an input, or renaming a job, breaks or renames the checks in every repo that uses it: update their `ci.yml` in the same round.
