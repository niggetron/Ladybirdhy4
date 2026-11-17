# Ladybird Android arm64 — automated builds

This repository automatically builds an **arm64-v8a release APK** of
[Ladybird](https://github.com/LadybirdBrowser/ladybird) from the upstream source
tree, signs it, and publishes it as a GitHub Release here.

> This is an **unofficial** build. Ladybird is pre-alpha and does not publish
> Android APKs of its own, so this repo builds it from upstream `master`.

## How it works

The workflow lives at `.github/workflows/build-arm64.yml`. On each run it:

1. Checks out `LadybirdBrowser/ladybird` (`master`) — it always builds the
   latest upstream source, not a vendored copy.
2. Sets up the same toolchain Ladybird's own CI uses: macOS 14, Xcode, LLVM,
   NDK r29, vcpkg, Rust, Java 17 and Gradle.
3. Patches the Android Gradle project to:
   - build **only** the `arm64-v8a` ABI, and
   - set `versionName` / `versionCode` for this build.
4. Builds the release APK with `./gradlew assembleRelease`.
5. Signs it with `apksigner` using the repository secrets.
6. Creates/updates a GitHub Release named `v<version>` and uploads the signed
   APK as `Ladybird-arm64-<version>.apk`.

Build caches (`ccache` and the vcpkg binary cache) are saved between runs so the
weekly rebuilds are much faster than the first one.

## Triggers

- **`push`** — any push to this repository starts a build.
- **Schedule** — `0 3 */7 * *` UTC (every 7 days, on the 1st, 8th, 15th, 22nd
  and 29th of the month) rebuilds from the latest upstream commit.
- **Manual** — Actions tab → *Ladybird Android arm64 (build + release)* →
  *Run workflow*. You can optionally override the version name.

> Note: GitHub can delay scheduled workflows under heavy load.

## Versioning

The auto-generated version name is:

```
1.0.<YYYYMMDD>.<run number>
```

For example `1.0.20261117.3`. The `versionCode` is the GitHub run number, so
every release has a strictly increasing code (important for Android updates).
The release tag is `v<version name>`.

## Signing

The APK is signed with a dedicated key stored in repository secrets:

- `SIGNING_KEYSTORE_B64`
- `SIGNING_STORE_PASSWORD`
- `SIGNING_KEY_ALIAS`
- `SIGNING_KEY_PASSWORD`

Because the same key is reused on every run, newer APKs can be installed as
updates over older ones. Keep these secrets — losing them means you cannot
publish a compatible update later.

## Build requirements / caveats

- Ladybird is a large native C++ browser engine. The first build can take a
  long time (potentially hours) and is limited to GitHub Actions' 6-hour job
  maximum.
- If the upstream build system changes in a way that breaks these commands,
  the workflow will need a small adjustment. Check the Actions log for the
  failing step.
