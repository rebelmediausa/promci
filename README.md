# Rebel Media PromCI

Build and publish Prometheus exporters with GitHub Actions.

[![Lint](https://github.com/rebelmediausa/promci/actions/workflows/lint.yml/badge.svg?branch=develop)](https://github.com/rebelmediausa/promci/actions/workflows/lint.yml)
[![CodeQL](https://github.com/rebelmediausa/promci/actions/workflows/codeql.yml/badge.svg?branch=develop)](https://github.com/rebelmediausa/promci/actions/workflows/codeql.yml)
[![OpenSSF Scorecard](https://api.scorecard.dev/projects/github.com/rebelmediausa/promci/badge)](https://scorecard.dev/viewer/?uri=github.com/rebelmediausa/promci)

PromCI is the Prometheus project's shared CI, packaged as GitHub composite
actions for Rebel Media exporters. It builds every platform with
[promu](https://github.com/prometheus/promu), passes the binaries between jobs
as artifacts, creates the GitHub release with tarballs and checksums, and can
push multi-architecture images to Docker Hub and Quay.io.

## How it works

The root action copies promci's own `actions/` directory into
`.github/promci/` in your workspace. Each capability is a composite action you
then call by local path:

```yaml
- uses: rebelmediausa/promci@<sha> # v1.3.1
- uses: ./.github/promci/actions/build
```

The copy comes from the action the runner has already downloaded, at exactly
the ref you pinned. **Pin promci to a full commit SHA** and every nested step
it runs is pinned with it: each third-party action inside promci is itself
pinned by SHA.

## Usage

A release workflow for an exporter: cross-build in six parallel jobs, create
the release, then publish tarballs and images.

```yaml
name: Release

on:
  push:
    tags: ["v*"]

permissions: {}

jobs:
  build:
    name: Build (${{ matrix.thread }})
    runs-on: ubuntu-latest
    permissions:
      contents: read
    strategy:
      matrix:
        thread: [0, 1, 2, 3, 4, 5]
    steps:
      - uses: actions/checkout@<sha> # v7.0.1
        with:
          fetch-depth: 0
          persist-credentials: false

      - uses: rebelmediausa/promci@<sha> # v1.3.1

      - uses: ./.github/promci/actions/build
        with:
          parallelism: 6
          thread: ${{ matrix.thread }}
          promu_codesign_binary: my_exporter

  publish:
    name: Publish
    needs: build
    runs-on: ubuntu-latest
    environment: release
    permissions:
      contents: write
    steps:
      - uses: actions/checkout@<sha> # v7.0.1
        with:
          fetch-depth: 0
          persist-credentials: false

      - uses: rebelmediausa/promci@<sha> # v1.3.1

      - uses: ./.github/promci/actions/publish_release
        with:
          image_name: my-exporter
          image_arch: linux/amd64,linux/arm64
          docker_hub_organization: rebelcore
          docker_hub_login: ${{ secrets.DOCKERHUB_USERNAME }}
          docker_hub_password: ${{ secrets.DOCKERHUB_TOKEN }}
          quay_io_organization: ""        # empty turns the Quay push off
          github_token: ${{ github.token }}
```

The Rebel Media exporter template (`go/exporter`) uses promci this way. It then
builds its own image with buildx instead of `publish_release`'s image step, so
exporter images get the same `X.Y.Z` / `X.Y` / `X` / `latest` tags as every
other Rebel Media image.

## Actions

### `setup_environment`

Prepares a runner for a promu build. Every other action calls it for you.

| Input | Default | Description |
|---|---|---|
| `enable_go` | `true` | Cache `~/.cache/go-build` and `~/go/pkg/mod`, keyed on `go.sum`. |
| `enable_npm` | `false` | Cache `~/.npm`, keyed on `web/ui/package-lock.json`. |
| `enable_docker_multibuild` | `false` | Set up QEMU and Docker Buildx for multi-arch images. |
| `memlimit_ratio` | `0.8` | Fraction of the container's memory limit (cgroup v1/v2, or MemTotal) to set as `GOMEMLIMIT`. |

It also installs promu, and exports `VERSION`, `REVISION`, `BRANCH` and `TAGS`
to the environment. promu stamps these into the binary. `VERSION` comes from
the tag when the commit is tagged (without the `v`), and from the `VERSION` file
otherwise.

### `build`

Cross-builds with promu and saves `.build/` as a workflow artifact for later
jobs.

| Input | Default | Description |
|---|---|---|
| `thread` | `3` (**required**) | Which slice of the platform list this job builds, `0` to `parallelism - 1`. |
| `parallelism` | `3` | How many jobs share the platform list. Match the matrix size. |
| `promu_config` | `.promu.yml` | promu config for the pure-Go build. Skipped if the file does not exist. |
| `promu_cgo_config` | `.promu-cgo.yml` | promu config for a cgo build. Skipped if the file does not exist. |
| `promu_opts` | *(empty)* | Extra promu options, separated by spaces. Each word is passed as one argument, and nothing is evaluated. |
| `promu_codesign_binary` | *(empty)* | Binary name to ad-hoc codesign in the darwin builds. |

### `publish_release`

Restores the build artifacts, builds tarballs, writes checksums, and uploads
them to the GitHub release for the tag with `promu release`. It then pushes
images to each registry whose organisation and login are both set.

| Input | Default | Description |
|---|---|---|
| `github_token` | *(empty)* | Token that can upload to the release: `${{ github.token }}` with `contents: write`. |
| `promu_config` | `.promu.yml` | promu config. |
| `image_name` | *(empty)* | Image name, without registry or organisation. |
| `image_tag` | the tag (`github.ref_name`) | Image tag. |
| `image_arch` | *(empty)* | Comma-separated buildx platforms, e.g. `linux/amd64,linux/arm64`. |
| `docker_hub_organization` | `prom` | Docker Hub namespace. Empty turns the Docker Hub push off. |
| `docker_hub_login` / `docker_hub_password` | *(empty)* | Docker Hub credentials. Use an access token, not a password. |
| `quay_io_organization` | `prometheus` | Quay.io namespace. Empty turns the Quay push off. |
| `quay_io_login` / `quay_io_password` | *(empty)* | Quay.io credentials. |

### `publish_release_images`

Called by `publish_release` for each registry. It builds and pushes one
multi-arch image with an SBOM, tagged `:<container_image_tag>`, and also
`:latest` unless the tag contains a hyphen (the semver pre-release marker, as in
`v1.2.0-rc.1`).

| Input | Default | Description |
|---|---|---|
| `registry` | *(empty)* | `docker.io` or `quay.io`. |
| `organization` | *(empty)* | Namespace in the registry. |
| `login` / `password` | *(empty)* | Registry credentials. |
| `container_image_name` | *(empty)* | Image name. |
| `container_image_tag` | `github.ref_name` | Image tag. |
| `container_image_arch` | `linux/amd64` | Comma-separated buildx platforms. |

### `publish_main` and `publish_main_images`

The same as the two release actions, for images built from a branch rather than
a tag (for example a `main` image on every merge). The inputs are the same,
without `image_tag` and `github_token`.

### `save_artifacts` / `restore_artifacts`

Save a directory as a uniquely named tar artifact, and restore every such
artifact in a later job. Tar keeps the file modes that `upload-artifact` would
otherwise lose.

| Input | Default | Description |
|---|---|---|
| `directory` (`save_artifacts`) | *(empty)* | Directory to save, e.g. `.build`. |

### `check_proto`

Installs `protoc` and runs `make proto`, then fails if the generated code
differs from what is committed.

| Input | Default | Description |
|---|---|---|
| `version` | `3.5.1` | protoc release to install. |

## Versioning

Releases follow semver. The major tag (`v1`) always points at the newest `v1.x.y`
release, so `@v1` follows fixes automatically, but a moving tag is exactly the
supply-chain risk that SHA pinning removes. **Pin to a commit SHA** with the
version as a comment (`@<sha> # v1.3.1`), and let Dependabot raise PRs when a
new release is out.

Release notes are on the [releases page](https://github.com/rebelmediausa/promci/releases)
and in [CHANGELOG.md](CHANGELOG.md).

## Security

promci runs inside release pipelines that hold publishing credentials, so:

- Every third-party action it uses is pinned to a commit SHA, and CI rejects an
  unpinned one.
- Inputs reach shell scripts only through quoted environment variables, never
  by interpolation. CI rejects `${{ inputs.* }}` inside a `run:` block.
- `develop` and `master` accept changes only by pull request, with passing
  checks and signed commits. Force pushes are blocked and nobody can bypass
  these rules. Only GitHub Actions can create or move release tags.
- Secret scanning with push protection, Dependabot, CodeQL and OpenSSF Scorecard
  are enabled.

Report vulnerabilities privately; see [SECURITY.md](SECURITY.md).

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md). In short: `feature/*` branches into
`develop`, signed and signed-off Conventional Commits, and only `develop` is
merged into `master`.

## License

Apache-2.0. See [LICENSE](LICENSE) and [NOTICE](NOTICE).
