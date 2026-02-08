# Rebel Media PromCI

Build and publish Prometheus exporters with GitHub Actions.

## Usage

In your exporter repository workflow:

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v6

      # Fetch this repo into .github/promci so you can use its composite actions.
      - uses: rebelmediausa/promci@v1

      - uses: ./.github/promci/actions/setup_environment
        with:
          enable_go: true

      - uses: ./.github/promci/actions/build
        with:
          thread: 1
          promu_codesign_binary: your_exporter_binary_name
```

## Included composite actions

- `./.github/promci/actions/setup_environment`: Go/NPM caching, sets `GOMEMLIMIT`, installs `promu`, and (optionally) enables Docker multi-arch build support.
- `./.github/promci/actions/check_proto`: Downloads `protoc` and runs `make proto`, then fails if `git diff` is non-empty.
- `./.github/promci/actions/build`: Runs `promu` crossbuild (and optional codesign) then saves `.build` as a workflow artifact.
- `./.github/promci/actions/save_artifacts` / `./.github/promci/actions/restore_artifacts`: Tar/upload build outputs and restore them in later jobs.
- `./.github/promci/actions/publish_main`: Pushes branch/tag images to Docker Hub and/or Quay (uses `publish_main_images`).
- `./.github/promci/actions/publish_release`: Creates GitHub releases via `promu` and pushes release images (uses `publish_release_images`).

## Image tagging behavior

`./.github/promci/actions/publish_release_images` always pushes `:<container_image_tag>`. It also pushes `:latest` unless the tag contains `-alpha`, `-beta`, or `-rc`.
