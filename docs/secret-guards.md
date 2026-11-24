# Credential skip guards

Every workflow step or composite action step that needs a secret, a repository variable, or an auth file is skipped when that value is empty or missing. An empty credential is treated as "this integration is not configured", not as a build error.

## How it works

- Reusable workflows copy `secrets.*` / `vars.*` into job-level `env`.
- Protected steps declare `if: env.NAME != ''`.
- Composite actions declare `if: inputs.name != ''`.
- A `::notice` step explains what was skipped and why.
- Steps without credentials (checkout, setup, cache) still run. The job itself is not failed.

Example:

```yaml
jobs:
  release:
    env:
      SERVICE_ACCOUNT_JSON: ${{ secrets.GOOGLE_PLAY_SERVICE_ACCOUNT_JSON }}
      TRACK: ${{ vars.TRACK }}
    steps:
      - name: Skip notice — missing secrets or vars
        if: env.SERVICE_ACCOUNT_JSON == '' || env.TRACK == ''
        run: echo "::notice title=Skipped release::A required secret or variable is empty; protected steps were skipped"
      - if: env.SERVICE_ACCOUNT_JSON != '' && env.TRACK != ''
        uses: example/release-action@v1
```

## Guarded integrations

| Flow | Required values |
| --- | --- |
| Google Play AAB release | `SHOREBIRD_TOKEN`, `GOOGLE_PLAY_SERVICE_ACCOUNT_JSON`, `TRACK` |
| Android release / patch | `SHOREBIRD_TOKEN` |
| Firebase Android distribution | `FIREBASE_ADMIN_SDK`, `SHOREBIRD_TOKEN` |
| GitHub APK release | `GITHUB_TOKEN` |
| iOS release, patch, dry-run | `SHOREBIRD_TOKEN`, `APP_STORE_CONNECT_ISSUER_ID`, `APP_STORE_CONNECT_API_KEY_ID`, `APP_STORE_CONNECT_API_PRIVATE_KEY`, `DISTRIBUTION_PRIVATE_KEY_BASE64` |
| Firebase Hosting deploy | `FIREBASE_SERVICE_ACCOUNT` (or the project-specific fallback), `PROJECT_ID` |
| Release cleanup | `GITHUB_TOKEN` and/or `FIREBASE_TOKEN` |
| Auto PR | `GITHUB_TOKEN` |

Composite actions apply the same rule internally:

- `shorebird-android-aab-release`: skips Play upload when the service account JSON or track is empty.
- `shorebird-android-apk-release`: skips GitHub release when the token is empty.
- `setup-ios-signing`: skips key-file creation, profile download and profile install when any App Store Connect input is empty.
- `shorebird-ios-dry-patch`: skips setup, dry-run and output calculation when any required input is empty. When it is skipped, downstream patch/release jobs are skipped too.
- `get-build-version`: does not call `shorebird releases list` without a token.

## Caller setup

- Pass secrets with `secrets: inherit` (or declare them explicitly). Secrets and variables come from the caller repository/environment, not from this repository.
- Define optional integrations only where they are used. Leaving a secret or variable empty disables that integration safely.
- Auth files (service account JSON, `.p8` key content, SSH private key) must contain the full file contents, not a path.
- The caller must still grant the permissions required by the reusable workflow; guards do not change permission requirements.

## Consumer repositories

The same guard pattern was applied to the standalone workflows in `muein_dash` (Firebase Hosting), `muein_orders` (signing/profiles) and `odhia_dash` (SFTP/SSH deploy).

## Branch status

The guard work is on these pushed branches:

- `ci_cd_pipeline`: `fix/skip-empty-secrets`
- `muein_dash`: `fix/skip-empty-secrets-consumers`, followed by `fix/workflow-permissions`, `ci/actionlint-check` and `ci/actionlint-push-dev`
- `muein_orders`: `fix/skip-empty-secrets-consumers`
- `odhia_dash`: `fix/skip-empty-secrets-consumers`

Every branch is below 400 changed lines and passes `actionlint` with `shellcheck`.

## Adding a new guarded step

1. Map the secret/variable to job-level `env`.
2. Add the notice step when more than one protected step is involved.
3. Add `if: env.NAME != ''` (or `inputs.name != ''` in a composite action) to the step that consumes the credential.
4. Validate with `actionlint` and `shellcheck` before committing.

Pin `@prod` to a commit SHA for maximum stability.
