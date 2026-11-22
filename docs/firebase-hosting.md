# Firebase Hosting reusable workflow

`50-firebase-hosting.yml` is the single source for building and deploying Flutter web to Firebase Hosting. The old `40-firebase-hosting-merge.yml` and `40-firebase-hosting-pull-request.yml` files were removed by the single-workflow refactor — do not re-add them.

It is called automatically by `00-main.yml` as `web_pipeline`; most projects only need to call `00-main.yml@prod`.

## What it does

- **Push**: deploys to the `live` channel; `ENV` dart-define = `github.ref_name`.
- **PR** (non-fork): deploys a preview channel + PR comment; `ENV` = `prod` when the base branch is `prod`, otherwise `dev`.
- **Environment**: `dev`/`prod` derived from the branch (push) or base branch (PR).
- **Flutter version**: read from the caller's `pubspec.yaml`.

## Caller setup

- Repository variables: `PROJECT_ID`, `TARGET`.
- Secret: `FIREBASE_SERVICE_ACCOUNT` (can be an environment secret).
- Environments: `dev` and `prod`.
- Permissions: `contents: read` (or `write` when calling `00-main`), `checks: write`, `pull-requests: write`.

### Option A — call 00-main (recommended)

Includes Auto PR, web hosting, and the platform pipelines:

```yaml
name: CI/CD Pipeline
on:
  push:
    branches: ['**']
  workflow_dispatch:
permissions:
  contents: write
  checks: write
  pull-requests: write
jobs:
  pipeline:
    uses: HishamKoptaN/ci_cd_pipeline/.github/workflows/00-main.yml@prod
    secrets: inherit
```

### Option B — web only

```yaml
name: Firebase Hosting
on:
  push:
    branches: [prod, dev]
  pull_request:
permissions:
  contents: read
  checks: write
  pull-requests: write
jobs:
  web:
    uses: HishamKoptaN/ci_cd_pipeline/.github/workflows/50-firebase-hosting.yml@prod
    secrets:
      FIREBASE_SERVICE_ACCOUNT: ${{ secrets.FIREBASE_SERVICE_ACCOUNT }}
```

`secrets: inherit` can be used instead for repositories under the same account/organization.

## Notes

- The `github` context inside the called workflow belongs to the caller, so variables, secrets and `GITHUB_TOKEN` come from the caller project.
- `checks: write` is required by the Firebase preview check; the caller job cannot grant less than the called workflow requests.
- Fork PRs are skipped by the workflow guard.
- Pin `@prod` to a commit SHA for maximum stability.
