# Connector images

Docker images of OpenCTI connectors from branches of
[FIying-Scotsman/connectors](https://github.com/FIying-Scotsman/connectors) that are
not merged upstream yet, published to the GitHub Container Registry.

These are pre-release builds of open pull requests, not official OpenCTI images.

## Images

Pull the latest build of each connector with:

```bash
docker pull ghcr.io/fiying-scotsman/connector-threatfox:pr-7404
docker pull ghcr.io/fiying-scotsman/connector-censys-enrichment:pr-7402
docker pull ghcr.io/fiying-scotsman/connector-google-ti-feeds:google-ti-feeds-enhanced-features
```

| Connector | Branch | Upstream PR | Tags |
|---|---|---|---|
| threatfox | `ThreatFox-Expanded` | [#7404](https://github.com/OpenCTI-Platform/connectors/pull/7404) | [list](https://github.com/FIying-Scotsman/connector-images/pkgs/container/connector-threatfox/versions) |
| censys-enrichment | `Censys-enrichment-expanded` | [#7402](https://github.com/OpenCTI-Platform/connectors/pull/7402) | [list](https://github.com/FIying-Scotsman/connector-images/pkgs/container/connector-censys-enrichment/versions) |
| google-ti-feeds | `google-ti-feeds-enhanced-features` | not opened yet | [list](https://github.com/FIying-Scotsman/connector-images/pkgs/container/connector-google-ti-feeds/versions) |

### Tags

Every build is pushed with these tags:

| Tag | Example | Points to |
|---|---|---|
| `pr-<number>` | `pr-7404` | Latest build of the pull request. Only when the image has an upstream PR. |
| `<branch>` (lower case) | `threatfox-expanded` | Latest build of the branch. Always present. |
| `sha-<commit>` | `sha-5133767` | One exact build, never moves. Use it to pin a deployment. |

There is no `latest` tag. The `pr-` and branch tags move to each new build; a `sha-`
tag stays on the build it was made for. The **list** links above show every tag of an
image with its build date. From a terminal, `skopeo list-tags docker://<image>` or
`crane ls <image>` lists them too.

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
