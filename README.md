# cloud-build-action

[![CI](https://github.com/capawesome-team/cloud-build-action/actions/workflows/ci.yml/badge.svg)](https://github.com/capawesome-team/cloud-build-action/actions/workflows/ci.yml)
[![GitHub Marketplace](https://img.shields.io/badge/Marketplace-Capawesome%20Cloud%20Build-blue?logo=github)](https://github.com/marketplace/actions/capawesome-cloud-build-action)
[![License](https://img.shields.io/github/license/capawesome-team/cloud-build-action)](./LICENSE)

GitHub Action to create a native app build on [Capawesome Cloud](https://cloud.capawesome.io/) Runners.

> [!NOTE]
> The `token` is sensitive and must be stored as an [encrypted secret](https://docs.github.com/en/actions/security-guides/encrypted-secrets) (e.g. `CAPAWESOME_TOKEN`) rather than hardcoded in the workflow. We recommend pinning the action to a fixed version (e.g. `@v0.1.0`) for reproducible builds, as no moving major-version tag is maintained.

## Usage

```yaml
- uses: capawesome-team/cloud-build-action@v0.1.0
  with:
    # The Capawesome Cloud app ID.
    # Required.
    appId: ''
    # Download the generated AAB file (Android only). Set to `true` or provide a file path.
    aab: ''
    # Download the generated APK file (Android only). Set to `true` or provide a file path.
    apk: ''
    # The name of the certificate to use for the build.
    certificate: ''
    # The name of the channel to deploy to (Web only). Cannot be combined with `destination`.
    channel: ''
    # The name of the destination to deploy to (Android/iOS only). Cannot be combined with `channel`.
    destination: ''
    # Exit immediately after creating the build without waiting for completion. Set to `true` to enable.
    detached: ''
    # The name of the environment to use for the build.
    environment: ''
    # The Git reference (branch, tag, or commit SHA) to build. Cannot be combined with `path` or `url`.
    gitRef: ''
    # Download the generated IPA file (iOS only). Set to `true` or provide a file path.
    ipa: ''
    # Path to local source files to upload. Must be a folder or a zip file. Cannot be combined with `gitRef` or `url`.
    path: ''
    # The platform for the build. Must be `ios`, `android`, or `web`.
    platform: ''
    # The build stack to use. Must be `macos-sequoia` or `macos-tahoe`.
    stack: ''
    # The Capawesome Cloud API token.
    # Required.
    token: ''
    # The type of build. iOS: `simulator`, `development`, `ad-hoc`, `app-store`, `enterprise`. Android: `debug`, `release`.
    type: ''
    # URL to a zip file to use as build source. Cannot be combined with `gitRef` or `path`.
    url: ''
    # Ad hoc environment variables, one `key=value` pair per line.
    variables: ''
    # Path to a file containing ad hoc environment variables in `.env` format.
    variableFile: ''
    # Download the generated ZIP file (Web only). Set to `true` or provide a file path.
    zip: ''
```

## Outputs

| Name          | Description                                           |
| ------------- | ----------------------------------------------------- |
| `buildId`     | The ID of the created build.                          |
| `buildNumber` | The build number.                                     |
| `buildUrl`    | The URL to the build in the Capawesome Cloud Console. |

## Example

```yaml
name: Build App
on:
  push:
    branches:
      - main
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      - name: Build App
        id: build
        uses: capawesome-team/cloud-build-action@v0.1.0
        with:
          appId: 'addb597c-9cbd-4cdc-bcc0-cd5c2234a03f'
          platform: 'android'
          type: 'release'
          gitRef: ${{ github.sha }}
          certificate: 'Release Keystore'
          aab: 'app-release.aab'
          token: ${{ secrets.CAPAWESOME_TOKEN }}
      - name: Upload artifact
        uses: actions/upload-artifact@v4
        with:
          name: app-release
          path: app-release.aab
```

## Notes

- **Waits by default**: The action waits for the build to complete and fails if the build fails. Native builds can take several minutes, so make sure the job timeout is high enough. Set `detached: true` to return immediately after the build is created.
- **Build source**: Provide exactly one of `gitRef`, `path`, or `url`. `gitRef` builds from the connected Git repository; `path` and `url` are experimental.
- **Deployment**: Use `channel` (Web only) or `destination` (Android/iOS only) to deploy after a successful build. They cannot be combined, and neither can be combined with `detached`.
- **Artifacts**: `apk`/`aab` (Android), `ipa` (iOS), and `zip` (Web) download the build artifact to the runner. They cannot be combined with `detached`.
- **Outputs require [`jq`](https://jqlang.github.io/jq/)**, which is preinstalled on GitHub-hosted runners.

## License

See [LICENSE](./LICENSE).
