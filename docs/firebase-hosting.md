# Firebase Hosting reusable workflows

Build and deploy Flutter web apps from any project without copying deployment steps into every repository. The logic lives in `HishamKoptaN/ci_cd_pipeline`; caller projects only keep a small trigger file.

## Available workflows

| Workflow | Trigger in caller | Result |
| --- | --- | --- |
| `40-firebase-hosting-merge.yml` | Push to `dev` / `prod` | Deploy to the live Hosting channel |
| `40-firebase-hosting-pull-request.yml` | Pull request | Deploy a preview channel + PR comment |

## Setup in every caller project

1. Add repository variable `PROJECT_ID` = Firebase project id (or pass `project-id` in `with`).
2. Add secret `FIREBASE_SERVICE_ACCOUNT` = Firebase service account JSON. It may be an environment secret: the reusable workflow sets `environment: dev/prod` itself, so environment secrets resolve correctly.
3. Create environments named `dev` and `prod`.
4. Grant the calling job at least `contents: write` (and `checks: write` / `pull-requests: write` for PR previews). Permissions can only be reduced inside a reusable workflow, never elevated.
5. Allow actions and reusable workflows from `HishamKoptaN/ci_cd_pipeline` in the repository Actions settings if required.

## Caller: deploy on merge

Create `.github/workflows/firebase-hosting-merge.yml`:

```yaml
name: Deploy to Firebase Hosting on merge
on:
  push:
    branches: [prod, dev]

permissions:
  contents: write
  pull-requests: write

jobs:
  deploy:
    uses: HishamKoptaN/ci_cd_pipeline/.github/workflows/40-firebase-hosting-merge.yml@prod
    secrets:
      FIREBASE_SERVICE_ACCOUNT: ${{ secrets.FIREBASE_SERVICE_ACCOUNT }}
```

Optional inputs under `with`: `flutter-version` (default `3.50.0`), `project-id` (default: caller variable `PROJECT_ID`), `channel-id` (default `live`).

## Caller: preview on pull request

Create `.github/workflows/firebase-hosting-pull-request.yml`:

```yaml
name: Deploy to Firebase Hosting on PR
on: pull_request

permissions:
  contents: write
  checks: write
  pull-requests: write

jobs:
  preview:
    uses: HishamKoptaN/ci_cd_pipeline/.github/workflows/40-firebase-hosting-pull-request.yml@prod
    secrets:
      FIREBASE_SERVICE_ACCOUNT: ${{ secrets.FIREBASE_SERVICE_ACCOUNT }}
```

Fork PRs are skipped automatically, matching the original workflow guard.

## Notes

- `ENV` passed to Dart is `github.ref_name` on push and `prod`/`dev` derived from the PR base branch.
- The `github` context inside a reusable workflow belongs to the caller, so `github.ref_name`, `github.base_ref`, `vars` and the `GITHUB_TOKEN` all come from the caller project.
- `secrets: inherit` can replace the explicit `secrets:` block for repositories under the same account/organization.
- Pin `@prod` to a commit SHA for maximum stability.
