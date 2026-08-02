# Release hygiene and ticket discipline

## Contents
- Version and release hygiene
- Ticket discipline

## Version and release hygiene

- Bump the version marker (versionCode/semver/build number) whenever a change
  affects shipped behavior, even a no-code-change redelivery — the point is
  recognizability between drops in analytics/crash tooling, not just marking
  that code changed.
- Bumping a version number is a text edit, not a build. Don't build or upload
  an artifact unless explicitly asked, and don't narrate a version bump as if
  it shipped something.
- Verify release-readiness with an actual build/test run, not a static config
  check — actually run the build and inspect the artifact, don't eyeball the
  config and assume.

## Ticket discipline

Create or update a tracking ticket for real changes, scoped to everything the
change actually touches — a change that ripples into a connected component
(e.g. a sidebar change that also touches a connected search box) belongs in
one ticket covering that whole scope, not fragmented across several
undersold ones. Don't let a ticket undersell what actually changed.
