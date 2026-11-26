# Free Disk Space

Composite action for reclaiming disk space on Ubuntu GitHub Actions runners before heavy Android/Flutter builds.

## Why

`ubuntu-latest` runners can run out of disk during Gradle/Shorebird Android builds. When the runner disk is exhausted, jobs can fail with errors such as `No space left on device`, including runner diagnostic write failures.

## Usage

```yaml
steps:
  - uses: HishamKoptaN/ci_cd_pipeline/.github/actions/core/free-disk-space@prod
  - uses: actions/checkout@v7
```

Use it before heavy build steps. It does not require checkout because it is referenced from the pipeline repository.

## Inputs

| Input | Default | Description |
| --- | --- | --- |
| `remove_dotnet` | `true` | Removes `/usr/share/dotnet`. |
| `remove_haskell` | `true` | Removes `/opt/ghc` and `/usr/local/.ghcup`. |
| `remove_boost` | `true` | Removes `/usr/local/share/boost`. |
| `remove_android_ndk` | `false` | Removes `/usr/local/lib/android/sdk/ndk`. Keep `false` unless the build does not need NDK. |
| `remove_docker_images` | `true` | Runs `docker image prune --all --force`. |
| `remove_tool_cache` | `false` | Removes `$AGENT_TOOLSDIRECTORY`. Enable only when later steps do not depend on hosted tool cache. |
| `min_free_gb` | `10` | Fails after cleanup if free space on `/` is below this GB threshold. Set `0` to disable. |

## Behavior

- Prints disk usage and key directory sizes before cleanup.
- Reclaims space based on enabled inputs.
- Prints disk usage after cleanup.
- Writes a small before/after summary to `GITHUB_STEP_SUMMARY` when available.
- Fails early if free space is below `min_free_gb`.

## NDK note

NDK removal is opt-in. Some Flutter/Android builds require the preinstalled Android NDK; removing it by default can break builds. Only set `remove_android_ndk: 'true'` when you have confirmed the workflow does not need it.
