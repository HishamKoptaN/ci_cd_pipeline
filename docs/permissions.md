# Permissions matrix

GitHub Actions does not let a reusable workflow grant itself more permissions than the caller job already has. The caller must grant at least the permissions declared by the called workflow; otherwise the run fails before any step starts.

## Rule

`caller job permissions >= reusable workflow permissions`

- If permissions are declared at the caller workflow level, every job inherits them.
- A job-level `permissions:` block overrides the workflow-level block for that job only.
- Omitted scopes are set to `none`, not to the repository default.
- `secrets: inherit` passes secrets only; it does not pass permissions.

## Required permissions

| Reusable workflow | Requires | Granted by `00-main.yml` | Notes |
| --- | --- | --- | --- |
| `00-main.yml` | `contents: write`, `checks: write`, `pull-requests: write` | Caller must grant all three | Top-level permissions apply to all its jobs unless overridden. |
| `auto-pr.yml` | `contents: write`, `pull-requests: write` | `auto_pr` job | Needed to create/reuse PRs. No extra secrets beyond `GITHUB_TOKEN`. |
| `50-firebase-hosting.yml` | `contents: read`, `checks: write`, `pull-requests: write` | `web_pipeline` job | `write` on `contents` is accepted because it is higher than `read`. Preview deployments need the PR scope. |
| `20-shorebird-layer.yml` | No explicit block; inherits caller | `prod_pipeline` job | Downstream store upload needs the inherited `contents: write`. |
| `50-distribution-layer.yml` | No explicit block; inherits caller | `dev_pipeline` job | GitHub Releases and Firebase distribution rely on the inherited permissions. |
| `00-actionlint.yml` | `contents: read` | Not called by `00-main` | Runs directly in each repository as the required lint check. |

## Caller examples

### Through `00-main` (recommended)

```yaml
permissions:
  contents: write
  checks: write
  pull-requests: write
jobs:
  pipeline:
    uses: HishamKoptaN/ci_cd_pipeline/.github/workflows/00-main.yml@prod
    secrets: inherit
```

### Direct hosting call

```yaml
permissions:
  contents: read
  checks: write
  pull-requests: write
jobs:
  web:
    uses: HishamKoptaN/ci_cd_pipeline/.github/workflows/50-firebase-hosting.yml@prod
    secrets: inherit
```

## Consumer status

- Callers of `00-main`: permissions are declared at workflow level and cover all three required scopes.
- Standalone Firebase Hosting workflows: both PR and deployment files declare the three scopes explicitly.
- Standalone SFTP/SSH deployment workflows: no GitHub API scopes are required beyond the default checkout permission; all authentication comes from secrets.
- Signing/profile workflows: use App Store Connect credentials, not GitHub API permissions.

## Verification

1. Open the caller workflow and inspect both workflow-level and job-level `permissions:`.
2. Compare with the table above.
3. Run `actionlint` locally or rely on the required `actionlint` check in each repository.
4. If a startup error mentions permissions, raise the caller scope — never add a broader scope inside the reusable workflow.

See [Credential skip guards](secret-guards.md) for how empty secrets/vars disable individual steps. Pin `@prod` to a commit SHA for maximum stability.
