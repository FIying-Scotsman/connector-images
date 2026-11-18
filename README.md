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
docker pull ghcr.io/fiying-scotsman/connector-csis:csis
```

Public images pull without logging in. A private image needs read access to its
package (package settings > Manage access) and a login with a classic personal access
token that has the `read:packages` scope:

```bash
echo "<token>" | docker login ghcr.io --username <github-user> --password-stdin
```

An image built while this repository is private starts private. A public image can
never be made private again: to hide one, delete its package and rebuild it.

| Connector | Branch | Upstream PR | Tags |
|---|---|---|---|
| threatfox | `ThreatFox-Expanded` | [#7404](https://github.com/OpenCTI-Platform/connectors/pull/7404) | [list](https://github.com/FIying-Scotsman/connector-images/pkgs/container/connector-threatfox/versions) |
| censys-enrichment | `Censys-enrichment-expanded` | [#7402](https://github.com/OpenCTI-Platform/connectors/pull/7402) | [list](https://github.com/FIying-Scotsman/connector-images/pkgs/container/connector-censys-enrichment/versions) |
| google-ti-feeds | `google-ti-feeds-enhanced-features` | not opened yet | [list](https://github.com/FIying-Scotsman/connector-images/pkgs/container/connector-google-ti-feeds/versions) |
| csis | `csis` | not opened yet | [list](https://github.com/FIying-Scotsman/connector-images/pkgs/container/connector-csis/versions) |

### Tags

| Tag | Example | Points to |
|---|---|---|
| `pr-<number>` | `pr-7404` | Latest default build of the pull request. Only when the image has an upstream PR. |
| `<branch>` (lower case) | `threatfox-expanded` | Latest default build of the branch. Always present. |
| `sha-<commit>-pycti-<version>` | `sha-5133767-pycti-7.261109.0` | One exact build: that commit with that pycti version. Never moves. Use it to pin a deployment. |

There is no `latest` tag. The `pr-` and branch tags move to each new default build. The
**list** links above show every tag of an image with its build date. From a terminal,
`skopeo list-tags docker://<image>` or `crane ls <image>` lists them too.

Builds with a [specific pycti version](#building-for-a-specific-opencti-version) get
their own tags with the version in them, such as `pr-7404-pycti-7.261023.1`, and leave
the plain tags alone.

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

Pushing a change to `images.json` builds every image. To rebuild after new commits on
a branch, run the workflow by hand:

1. Open [Actions > Build connector images](https://github.com/FIying-Scotsman/connector-images/actions/workflows/build-images.yml).
2. Click **Run workflow**.
3. Fill in:
   - `only`: one image name from `images.json`, such as `threatfox`, or empty for all.
     A named image is built even if its pull request is merged (see
     [When a pull request is merged](#when-a-pull-request-is-merged)).
   - `pycti_version`: empty for the default build, or a version (see below).
4. Click **Run workflow** to start it.

Images are built for `linux/amd64`.

### pycti version

The connector branches install `connectors-sdk` from upstream `master`, which pins one
exact pycti version. A branch pinning another version cannot be installed with it, so
the workflow replaces the branch pin:

- **Default build** (`pycti_version` empty): uses the pycti version of `connectors-sdk`
  on upstream `master`. When upstream bumps pycti, the next build follows, with nothing
  to change here. These builds move the `pr-` and branch tags.
- **Specific version** (`pycti_version` set): see below.

### Building for a specific OpenCTI version

pycti versions follow the OpenCTI releases. To build for a platform that is not on the
latest release, enter its version as `pycti_version`, for example `7.261023.1`. The
workflow then:

- pins pycti to that version, and `connectors-sdk` to the upstream release tag of the
  same name, whose own pycti pin matches. A version with no such tag stops the build
  with an error: use an OpenCTI release version.
- tags the image `pr-<number>-pycti-<version>`, `<branch>-pycti-<version>` and
  `sha-<commit>-pycti-<version>`. The plain `pr-` and branch tags keep pointing at
  the default build.

A branch written against a newer `connectors-sdk` than that release may not build or
run with it: check the connector on the target platform before handing it over.

## Cleanup

GitHub never deletes images by itself, and every rebuild leaves the previous build
behind. The **Clean up old images** workflow runs every Monday at 04:00 UTC and, for
each image in `images.json`, keeps:

- the current builds: every version a `pr-` or branch tag points to, including
  `-pycti-<version>` builds of a specific OpenCTI version;
- the 2 newest older builds (versions only tagged `sha-...`), for rollback. Change
  `KEEP_BUILDS` in the workflow to keep more or fewer: private images count against
  the 500 MB of storage of the free plan, public ones are free;
- the manifests those versions reference. GitHub lists them as untagged versions, but
  the tagged image breaks without them.

Everything else, including untagged versions, is deleted. A deployment pinned to a
`sha-` tag older than the 2 newest builds stops pulling, so hand over a newer build
before it falls out.

To preview, run the workflow by hand with `dry_run` ticked (the default): the run
summary lists what it would delete. Untick it to delete now instead of waiting for
Monday.

## When a pull request is merged

The official OpenCTI image then replaces this one. Both workflows check the `pr` of
each entry upstream:

- **From the merge:** the image is not built any more. Existing tags keep working, so
  the vendor can switch to the official image (`opencti/connector-<name>`) at their
  own pace.
- **30 days after the merge:** the cleanup deletes the whole package. Change
  `RETIRE_DAYS` in the cleanup workflow for a longer or shorter grace period. GitHub
  keeps a deleted package restorable for 30 days, from the package settings of your
  profile.
- **The entry stays in `images.json`.** Leave it there: it is skipped, so changing
  `images.json` to add another image does not rebuild it.

To ship a fix on top of the merged code:

- **Before a new pull request:** push the fix to the branch, then run the build by
  hand with the image name in `only`. Naming an image builds it even when its pull
  request is merged; it updates the branch and `sha-` tags (and the old `pr-` tag).
  Running the build for all images still skips it. The cleanup keeps retiring it 30
  days after the old merge, so set `pr` to the new pull request once it is open.
- **With a new pull request:** set the entry's `pr` to the new number (and `branch` if
  it changed). The image builds again, under a new `pr-<number>` tag, and its package
  is recreated if the cleanup had deleted it. That merge retires it again the same
  way.

An image that never had a pull request (no `pr`) always builds: nothing is checked
upstream.

An entry without `pr` is never retired automatically: remove it from `images.json`
and delete its package by hand from the package settings.
