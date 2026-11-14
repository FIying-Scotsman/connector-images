# Connector images

Docker images of OpenCTI connectors from branches of
[FIying-Scotsman/connectors](https://github.com/FIying-Scotsman/connectors) that are
not merged upstream yet, published to the GitHub Container Registry.

These are pre-release builds of open pull requests, not official OpenCTI images.

## Images

| Image | Branch | Upstream PR |
|---|---|---|
| `ghcr.io/fiying-scotsman/connector-threatfox` | `ThreatFox-Expanded` | [#7404](https://github.com/OpenCTI-Platform/connectors/pull/7404) |
| `ghcr.io/fiying-scotsman/connector-censys-enrichment` | `Censys-enrichment-expanded` | [#7402](https://github.com/OpenCTI-Platform/connectors/pull/7402) |
| `ghcr.io/fiying-scotsman/connector-google-ti-feeds` | `google-ti-feeds-enhanced-features` | not opened yet |

Tags:

- `pr-<number>`: latest build of the pull request
- `<branch>` (lower case): latest build of the branch
- `sha-<commit>`: one exact build, for pinning a deployment

```bash
docker pull ghcr.io/fiying-scotsman/connector-threatfox:pr-7404
```

Configuration is the same as the upstream connector: see the README of the connector
on its branch.

## Adding an image

Add an entry to `images.json`:

```json
{
  "name": "censys-collections",
  "branch": "Censys-Collections",
  "path": "external-import/censys-collections",
  "pr": 1234
}
```

- `name`: image name, published as `ghcr.io/fiying-scotsman/connector-<name>`. Keep it
  once the image is in use, or deployments pulling it must change.
- `branch`: branch of the connectors fork to build.
- `path`: connector directory in that branch, holding its `Dockerfile`.
- `pr`: upstream pull request number. Optional: leave it out (or `null`) for a branch
  without a pull request; the image then only gets the branch and `sha-` tags. Add it
  once the pull request is open to also tag `pr-<number>`.

## Building

`images.json` lists the images. Pushing a change to it builds every image. To rebuild
after new commits on a branch, run **Actions > Build connector images > Run workflow**:

- `only`: one image name, or empty for all
- `pycti_version`: the pycti version matching the OpenCTI platform the image will
  connect to, or empty to keep the branch pin

Images are built for `linux/amd64`.
