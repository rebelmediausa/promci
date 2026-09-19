# Changelog

## 1.3.1

- Fix: `setup_environment` used the branch name as the version on a branch
  run (`workflow_dispatch` on a branch), and rewrote `VERSION` with it. The ref
  name is now used only when the run was started by a tag, and `VERSION` is
  rewritten only when the version differs.
- Fix: the build step no longer fails when the commit is on no branch, as a
  pull request's merge commit is.
- Security: every input reaches the shell through a quoted environment variable
  instead of being interpolated into the script (`build`, `check_proto`,
  `publish_release`, `publish_release_images`, `save_artifacts`,
  `setup_environment`). `promu_opts` is split on whitespace into separate
  arguments and never evaluated.
- Releases now also publish a GitHub release with the CHANGELOG notes.
- Repository: gitflow (develop is the default branch), rulesets, CodeQL,
  Scorecard, dependency review, and a Lint check that rejects unpinned actions
  and interpolated inputs.

## 1.3.0

- Update the actions promci runs to their current majors: checkout v7.0.1,
  cache v6.1.0, upload-artifact v7.0.1, download-artifact v8.0.1,
  github-script v9.0.0, docker/login-action v4.6.0,
  docker/build-push-action v7.4.0, docker/setup-buildx-action v4.4.1
- Dependabot: one grouped weekly PR for minor and patch updates (Monday 06:00
  America/New_York), majors individually, and a 7-day cooldown on new releases
- Tagging skips a merge that does not change VERSION instead of failing

## 1.2.0

- Pin every nested action to a full commit SHA (same major versions as before)
- Copy the actions from the downloaded action instead of checking out `v1.1.0`, so pinning promci pins the code it runs
- Dependabot now watches every action directory
- README: `latest` is skipped for any pre-release tag, which is what the code does

## 1.1.0

- fix univerasl building and testing

## 1.0.6

- Allow for beta alpha and rc builds
- Code cleanup

## 1.0.5

- Add env to build

## 1.0.1

- Add some components from the CircleCI build

## 1.0.0

- Initial Release
