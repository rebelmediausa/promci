# Contributing

Rebel Media uses GitHub to manage reviews of pull requests.

* If you have a trivial fix or improvement, go ahead and create a pull request,
  addressing (with `@...`) the maintainer of this repository (see
  [MAINTAINERS.md](MAINTAINERS.md)) in the description of the pull request.

* Sign your work to certify that your changes were created by yourself, or you
  have the right to submit it under our license. Read
  https://developercertificate.org/ for all details and append your sign-off to
  every commit message like this:

        Signed-off-by: Random J Developer <example@example.com>

  Commits must also be GPG-signed; the branch rules reject unsigned commits.
  `git commit -s -S` does both.

## Branching

`develop` is the default branch and where pull requests are opened. Nobody
pushes to it directly. `master` only ever receives a merge from `develop`, and a
merge to `master` with a new `VERSION` publishes a release. Work happens on
`feature/<short-description>` branches taken from `develop`.

## Commit messages

[Conventional Commits](https://www.conventionalcommits.org/):
`<type>(<optional scope>): <summary>` with type `feat`, `fix`, `refactor`,
`docs`, `ci`, `chore` or `deps`. The pull request title follows the same rule,
because it becomes the squash commit.

## Rules for action code

These are what the Lint check enforces, and why:

- **Pin every external action to a full commit SHA** with the version as a
  comment: `uses: actions/checkout@<40-hex> # v7.0.1`. A tag is a moving pointer
  someone else controls, and this action runs inside other people's release
  pipelines. Dependabot keeps the pins current.
- **Never interpolate an input into a script.** Pass it through `env:` and quote
  it:

  ```yaml
  - shell: bash
    env:
      NAME: ${{ inputs.name }}
    run: git config user.name "$NAME"
  ```

  `${{ inputs.name }}` inside `run:` is pasted into the script before the
  shell sees it, so a value with a space breaks and a value with `$(...)` runs.
- Start every multi-line script with `set -euo pipefail`.
- Document every input and output in the README in the same change.

## Releasing

1. On a `feature/release-x.y.z` branch, set `VERSION` to `x.y.z` and add a
   `## x.y.z` section to `CHANGELOG.md`. Merge to `develop`.
2. Test from a real consumer workflow pinned to the develop commit SHA.
3. Open a pull request from `develop` to `master` and merge it.
4. CI tags `vx.y.z`, publishes the GitHub release and moves `v1`. Nobody tags
   by hand: version tags are immutable once created, so a tag made by mistake
   cannot be repointed.
