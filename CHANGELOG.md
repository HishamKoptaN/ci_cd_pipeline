# Changelog

All notable changes to the CI/CD framework in this repository.

## [Unreleased]

### Added

- Credential skip guards for reusable workflows and composite actions. Empty secrets, variables or auth files now skip protected steps with a visible notice instead of failing the run. See [docs/secret-guards.md](docs/secret-guards.md).
- Permissions matrix documentation for reusable workflows and callers. See [docs/permissions.md](docs/permissions.md).
- `actionlint` check in `muein_dash` on every PR, on pushes to `dev` and on manual runs.
- Branch status summary in `docs/secret-guards.md`.

### Changed

- Google Play, Android, iOS, Firebase Hosting, Auto PR, cleanup and distribution flows map secrets/vars to job-level `env` and guard the steps that consume them.
- Composite actions guard Play upload, GitHub release, iOS signing and dry-run steps.
- Firebase Hosting skips both build and deploy when the service account or `PROJECT_ID` is empty.
- `README.md` links the new credential and permissions documents.

### Fixed

- Corrected the trigger branch name in `00-main.yml`.
- Declared explicit permissions in the standalone Firebase Hosting deployment workflow in `muein_dash`.
- `get-build-version` no longer queries Shorebird without a token.
- Same guard pattern applied to standalone workflows in `muein_orders` (signing/profiles) and `odhia_dash` (SFTP/SSH deploy).

### Branch status

- `ci_cd_pipeline`: `fix/skip-empty-secrets`, `docs/skip-empty-secrets`, `fix/main-branch-typo`, `docs/permissions-matrix`, `docs/unified-readme-links`, `docs/permissions-actionlint-note`, `docs/guards-branch-status`
- `muein_dash`: `fix/skip-empty-secrets-consumers`, `fix/workflow-permissions`, `ci/actionlint-check`, `ci/actionlint-push-dev`
- `muein_orders`: `fix/skip-empty-secrets-consumers`
- `odhia_dash`: `fix/skip-empty-secrets-consumers`

Every branch is below 400 changed lines and passes `actionlint` with `shellcheck`.
