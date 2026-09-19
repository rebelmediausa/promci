<!--
Pull requests go feature/<name> -> develop. Only develop is ever merged into
master, and a merge to master with a new VERSION publishes a release.
-->

## Summary

<!-- What does this change, and why? Link the issue it closes. -->

Closes #

## Checklist

- [ ] Title is a Conventional Commit (`feat: ...`, `fix: ...`, `ci: ...`)
- [ ] Every commit is signed off and GPG-signed (`git commit -s -S`)
- [ ] Every `uses:` is pinned to a full commit SHA with a `# vX.Y.Z` comment
- [ ] Inputs reach shell scripts through `env:` and are quoted, never `${{ inputs.x }}` inside `run:`
- [ ] README updated for any new or changed input or output
- [ ] **If users will notice this change**, an entry under the next version in `CHANGELOG.md`

### If this is a release (develop -> master)

- [ ] `VERSION` bumped, and `CHANGELOG.md` has a `## <VERSION>` section
- [ ] Tried from a real consumer workflow pinned to this branch's commit

## Notes for the reviewer
