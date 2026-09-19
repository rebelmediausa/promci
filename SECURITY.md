# Security Policy

The Rebel Media security policy, which takes precedence over anything here, is
published at <https://docs.rebelmedia.io/security>.

## Supported versions

Only the latest release of the current major version (`v1`) receives fixes.
Pinning to a commit SHA is recommended; update the pin when a fix is released.

## Reporting a vulnerability

Report privately. Do not open a public issue.

Use [GitHub private vulnerability reporting](https://github.com/rebelmediausa/promci/security/advisories/new),
or email <security@rebelmedia.io>.

A useful report says what the problem is, how to reproduce it, and what an
attacker gains: for an action, usually which workflow configuration lets
untrusted input reach a shell, a token, or a secret.

## What happens next

1. We acknowledge the report within 5 business days.
2. We confirm the issue and agree the severity, normally within 10 business
   days.
3. We release a fix. Coordinated disclosure is 90 days from acknowledgement, or
   sooner once a release is out.
4. We publish a GitHub Security Advisory and credit you, unless you would
   prefer otherwise.

## How this repository is protected

- `develop` and `master` accept changes only through pull requests with passing
  checks and signed commits; force pushes and deletion are blocked, with no
  bypass for anyone.
- Version tags (`vX.Y.Z`) can never be moved or deleted, and releases are
  immutable. The major tag (`v1`) cannot be deleted; the release workflow moves
  it to each new release.
- Every external action is pinned by commit SHA, enforced by CI and by the
  repository's Actions policy.
- Secret scanning with push protection, Dependabot alerts and security updates,
  CodeQL and OpenSSF Scorecard are enabled.
