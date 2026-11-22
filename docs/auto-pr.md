# Auto PR reusable workflow

`auto-pr.yml` creates (or reuses) a pull request to `dev` when a branch is pushed. It is called by `00-main.yml` as `auto_pr` for every branch except `dev`/`prod`. The old `60-auto-pr.yml` file was superseded by this single file.

## Caller setup

- Permissions on the calling job: `contents: write` and `pull-requests: write`.
- No extra secrets: the caller's `GITHUB_TOKEN` is used.
- When calling `00-main.yml`, its `auto_pr` job already grants the needed permissions, but the outer caller job must still allow `pull-requests: write`.

## Option A — through 00-main (recommended)

```yaml
name: CI/CD Pipeline
on:
  push:
    branches: ['**']
permissions:
  contents: write
  checks: write
  pull-requests: write
jobs:
  pipeline:
    uses: HishamKoptaN/ci_cd_pipeline/.github/workflows/00-main.yml@prod
    secrets: inherit
```

## Option B — Auto PR only

```yaml
name: Auto PR
on:
  push:
    branches-ignore: [prod, dev]
permissions:
  contents: write
  pull-requests: write
jobs:
  auto-pr:
    uses: HishamKoptaN/ci_cd_pipeline/.github/workflows/auto-pr.yml@prod
```

## Notes

- The called workflow also guards against `dev`/`prod`.
- Source branch = pushed branch, destination = `dev`; title and body are defined in `auto-pr.yml`.
- If a PR already exists, the action returns it instead of creating a duplicate.
- `branches-ignore` must stay in the caller workflow; triggers are not inherited.
- Pin `@prod` to a commit SHA for maximum stability.
