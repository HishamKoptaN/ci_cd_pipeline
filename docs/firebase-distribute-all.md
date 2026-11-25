# Firebase Distribute All Platforms

`50-firebase-distribute-all.yml` is a thin entry point that calls four reusable flows in parallel:

- `50-distribute-android.yml` — Android APK through Firebase App Distribution.
- `20-deploy-google-play.yml` — Android AAB upload to Google Play.
- `30-ios-release.yml` — iOS release build.
- `50-firebase-hosting.yml` — Web deploy to Firebase Hosting.

Call it through `00-main.yml` or as a reusable workflow. It requires the same inputs/secrets as the individual flows (e.g. `SHOREBIRD_TOKEN`, `FIREBASE_ADMIN_SDK`, `FIREBASE_SERVICE_ACCOUNT`, `GOOGLE_PLAY_SERVICE_ACCOUNT_JSON`).

## Flavor variables

The reusable workflows accept an optional `flavor` input and resolve it in this order:

1. Explicit `flavor` passed by the caller.
2. Environment variable `vars.PER_FLAVOR`.
3. `github.ref_name` as the fallback.

Use this when different projects want different flavors. Example vars:

```yaml
# green_souq
PROJECT_ID: greensoouq-dev
PER_FLAVOR: dev

# muein_orders
PER_FLAVOR: prod
```

See [Permissions matrix](permissions.md) for the required permissions.

## Consumers using PER_FLAVOR

- `green_file_flutter`: `deploy.yml` sets `env.PER_FLAVOR` from `vars.PER_FLAVOR || github.ref_name` and uses it in `flutter build web`.
- `muein_dash`: the merge/PR hosting workflows set `env.PER_FLAVOR` from `vars.PER_FLAVOR` (merge uses `github.ref_name`, PR uses `github.base_ref`) and use it in `flutter build web`.

The workflow grants `contents: write`, `checks: write` and `pull-requests: write`; callers must grant at least these. See [Permissions matrix](permissions.md).
