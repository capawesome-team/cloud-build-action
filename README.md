# Capawesome Cloud Build Action for GitHub Actions

[![CI](https://github.com/capawesome-team/cloud-build-action/actions/workflows/ci.yml/badge.svg)](https://github.com/capawesome-team/cloud-build-action/actions/workflows/ci.yml)
[![GitHub Marketplace](https://img.shields.io/badge/Marketplace-Capawesome%20Cloud%20Build-blue?logo=github)](https://github.com/marketplace/actions/capawesome-cloud-build-action-for-github-actions)
[![License](https://img.shields.io/github/license/capawesome-team/cloud-build-action)](./LICENSE)

GitHub Action to create a native app build on [Capawesome Cloud](https://cloud.capawesome.io/) Runners.

> [!NOTE]
> The `token` is sensitive and must be stored as an [encrypted secret](https://docs.github.com/en/actions/security-guides/encrypted-secrets) (e.g. `CAPAWESOME_TOKEN`) rather than hardcoded in the workflow. We recommend pinning the action to a fixed version (e.g. `@v0.1.2`) for reproducible builds, as no moving major-version tag is maintained.

## Related Actions

- [Capawesome Cloud Live Update Action](https://github.com/capawesome-team/cloud-live-update-action) — Deploy a Capacitor Live Update to the Capawesome Cloud.

## Usage

```yaml
- uses: capawesome-team/cloud-build-action@v0.1.2
  with:
    # The Capawesome Cloud app ID.
    # Required.
    appId: ''
    # The Capawesome Cloud API token.
    # Required.
    token: ''
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
    # Create a public share link for the build. Not compatible with `detached`. Set to `true` to enable.
    share: ''
    # Additional information shown on the public share page, e.g. what to test.
    shareDescription: ''
    # The number of days until the share link expires.
    shareExpiresInDays: ''
    # The build stack to use. Must be `macos-sequoia` or `macos-tahoe`.
    stack: ''
    # The type of build. iOS: `simulator`, `development`, `ad-hoc`, `app-store`, `enterprise`. Android: `debug`, `release`.
    type: ''
    # URL to a zip file to use as build source. Cannot be combined with `gitRef` or `path`.
    url: ''
    # Path to a file containing ad hoc environment variables in `.env` format.
    variableFile: ''
    # Ad hoc environment variables, one `key=value` pair per line.
    variables: ''
    # Download the generated ZIP file (Web only). Set to `true` or provide a file path.
    zip: ''
```

## Outputs

| Name             | Description                                                   |
| ---------------- | ------------------------------------------------------------ |
| `buildId`        | The ID of the created build.                                 |
| `buildNumber`    | The build number.                                            |
| `buildUrl`       | The URL to the build in the Capawesome Cloud Console.        |
| `shareWebUrl`    | The URL to the public share page for the build.              |
| `shareQrCodeUrl` | The URL to a QR code image (PNG) encoding the public share page. |
| `shareExpiresAt` | The ISO 8601 timestamp when the share link expires, if set.  |

## Example

### Create a build

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
        uses: capawesome-team/cloud-build-action@v0.1.2
        with:
          appId: 'addb597c-9cbd-4cdc-bcc0-cd5c2234a03f'
          platform: 'android'
          type: 'release'
          gitRef: ${{ github.sha }}
          certificate: 'Release Keystore'
          aab: 'app-release.aab'
          token: ${{ secrets.CAPAWESOME_TOKEN }}
      - name: Print build outputs
        run: |
          echo "Build ID: ${{ steps.build.outputs.buildId }}"
          echo "Build number: ${{ steps.build.outputs.buildNumber }}"
          echo "Build URL: ${{ steps.build.outputs.buildUrl }}"
      - name: Upload artifact
        uses: actions/upload-artifact@v4
        with:
          name: app-release-${{ steps.build.outputs.buildNumber }}
          path: app-release.aab
```

### Share a build in a pull request

Set `share: true` to create a public share link, then post the QR code and share link as a pull request comment so testers can install the build directly from their phone.

```yaml
name: Share Build
on:
  pull_request:
jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      pull-requests: write
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      - name: Build App
        id: build
        uses: capawesome-team/cloud-build-action@v0.1.2
        with:
          appId: 'addb597c-9cbd-4cdc-bcc0-cd5c2234a03f'
          platform: 'android'
          type: 'release'
          gitRef: ${{ github.sha }}
          share: true
          shareDescription: 'Please test the new checkout flow.'
          shareExpiresInDays: 7
          token: ${{ secrets.CAPAWESOME_TOKEN }}
      - name: Comment share link on PR
        uses: actions/github-script@v7
        with:
          script: |
            const sha = context.payload.pull_request?.head?.sha || context.sha;
            const commitUrl = `${context.serverUrl}/${context.repo.owner}/${context.repo.repo}/commit/${sha}`;
            const expiresAt = '${{ steps.build.outputs.shareExpiresAt }}'.slice(0, 10);
            const body = [
              '### 📱 Android build ready to test',
              '',
              '| Build | Commit | Expires |',
              '| --- | --- | --- |',
              `| [#${{ steps.build.outputs.buildNumber }}](${{ steps.build.outputs.buildUrl }}) | [\`${sha.slice(0, 7)}\`](${commitUrl}) | ${expiresAt} |`,
              '',
              `[<img src="${{ steps.build.outputs.shareQrCodeUrl }}" width="140" alt="QR code" />](${{ steps.build.outputs.shareWebUrl }})`,
              '',
              `**[Open share page →](${{ steps.build.outputs.shareWebUrl }})**`,
            ].join('\n');
            await github.rest.issues.createComment({
              owner: context.repo.owner,
              repo: context.repo.repo,
              issue_number: context.issue.number,
              body,
            });
```

## Notes

- **Waits by default**: The action waits for the build to complete and fails if the build fails. Native builds can take several minutes, so make sure the job timeout is high enough. Set `detached: true` to return immediately after the build is created.
- **Build source**: Provide exactly one of `gitRef`, `path`, or `url`. `gitRef` builds from the connected Git repository; `path` and `url` are experimental.
- **Deployment**: Use `channel` (Web only) or `destination` (Android/iOS only) to deploy after a successful build. They cannot be combined, and neither can be combined with `detached`.
- **Artifacts**: `apk`/`aab` (Android), `ipa` (iOS), and `zip` (Web) download the build artifact to the runner. They cannot be combined with `detached`.
- **Sharing**: Set `share: true` to create a public share page for the build. Use `shareDescription` to add testing notes and `shareExpiresInDays` to expire the link. It cannot be combined with `detached`. The `shareQrCodeUrl` output is a public PNG image you can embed directly in Markdown or post to services like Slack.
- **Outputs require [`jq`](https://jqlang.github.io/jq/)**, which is preinstalled on GitHub-hosted runners.

## License

See [LICENSE](./LICENSE).
