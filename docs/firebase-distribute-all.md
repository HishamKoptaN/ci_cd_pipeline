# Firebase Distribute All Platforms

`50-firebase-distribute-all.yml` is a thin entry point that calls four reusable flows in parallel:

- `50-distribute-android.yml` — Android APK through Firebase App Distribution.
- `20-deploy-google-play.yml` — Android AAB upload to Google Play.
- `30-ios-release.yml` — iOS release build.
- `50-firebase-hosting.yml` — Web deploy to Firebase Hosting.

Call it through `00-main.yml` or as a reusable workflow. It requires the same inputs/secrets as the individual flows (e.g. `SHOREBIRD_TOKEN`, `FIREBASE_ADMIN_SDK`, `FIREBASE_SERVICE_ACCOUNT`, `GOOGLE_PLAY_SERVICE_ACCOUNT_JSON`).

The workflow grants `contents: write`, `checks: write` and `pull-requests: write`; callers must grant at least these. See [Permissions matrix](permissions.md).
