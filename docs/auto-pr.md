# Auto PR reusable workflow

Opens (or reuses) a pull request to `dev` whenever a branch is pushed. The workflow lives in `HishamKoptaN/ci_cd_pipeline`; each project only needs a small trigger file.

## Caller setup

Create `.github/workflows/auto-pr.yml` in your project:

```yaml
name: Auto PR
on:
  push:
    branches-ignore:
      - prod
      - dev

permissions:
  contents: write
  pull-requests: write

jobs:
  pull-request:
    uses: HishamKoptaN/ci_cd_pipeline/.github/workflows/60-auto-pr.yml@prod
```

No extra secrets are needed: the called workflow uses the caller's `GITHUB_TOKEN`, and permissions must be granted on the calling job (a called workflow cannot elevate them).

## Optional inputs

| Input | Default | Purpose |
| --- | --- | --- |
| `destination-branch` | `dev` | Target branch of the PR. |
| `source-branch` | Pushed branch (`github.ref_name`) | Source branch override. |
| `destination-repository` | Checked-out repository | Open the PR in another `owner/repo`. |
| `pr-title` | `Automated PR from <ref>` | Custom title. |
| `pr-body` | Automated PR notice with branch name | Custom body. |

Example with overrides:

```yaml
jobs:
  pull-request:
    uses: HishamKoptaN/ci_cd_pipeline/.github/workflows/60-auto-pr.yml@prod
    with:
      destination-branch: main
      pr-title: "Auto PR: ${{ github.ref_name }} → main"
      pr-body: "Please review this branch."
```

## Notes

- The `branches-ignore` trigger must stay in the caller workflow; triggers cannot be inherited. The reusable workflow also guards against `dev`/`prod` as a safety net.
- If a PR already exists for the branch, the action returns it instead of creating a duplicate.
- The workflow is push-only, so fork PRs never reach it.
- Pin `@prod` to a commit SHA for maximum stability.
