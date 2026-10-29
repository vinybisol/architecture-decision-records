---
number: 18
title: Build disposable pre-release versions from every integration into develop
date: 2026-10-29
status: accepted
tags:
- versioning
- pre-release
- ci-cd
- artifacts
links:
- target: 13
  kind: relatesto
- target: 14
  kind: relatesto
- target: 16
  kind: relatesto
- target: 19
  kind: relatesto
---

# 18. Build disposable pre-release versions from every integration into develop

Date: 2026-10-29

## Status

Accepted

## Context

Changes merged into `develop` (ADR 15) need to be built and deployed to a development environment before a release exists. Those builds also need a version, so that people and logs can tell which build is running and which source revision it came from.

Computing the next release version at integration time is not possible in this process, because the increment is only decided at release time from the changelog (ADR 13). Pre-release builds therefore have to derive their version from information that already exists: the latest release tag (ADR 14) reachable from `develop`.

This decision is independent of language and CI/CD platform.

## Decision

Every push to `develop`, including merged pull requests, triggers a pre-release build.

### Version

The pre-release version is:

```text
<latest reachable release>-dev.<build number>+<short revision>
```

- `<latest reachable release>` is the most recent `vX.Y.Z` tag that is an ancestor of the built revision, without the `v` prefix. When no tag exists, `0.0.0` is used.
- `<build number>` is a monotonically increasing number provided by the CI/CD platform, which makes each pre-release unique.
- `<short revision>` is the abbreviated source revision, recorded as build metadata.

For example, `0.2.0-dev.15+a1b2c3d`. The version is injected into the build in the same way as a release version (ADR 14); nothing is written to project files.

The build must have access to the repository's full history and tags. If a release tag is not reachable from `develop`, the back-merge required by ADR 16 is missing.

### Disposable artifacts

Pre-release artifacts are stored for a short period, by default seven days, and may be deployed to development environments. They do not create tags or published releases and are never promoted to production.

### Precedence

By Semantic Versioning precedence, `0.2.0-dev.N` is lower than `0.2.0` even though it was built after the `0.2.0` release. This is accepted because pre-release builds are never promoted or compared against releases.

## Consequences

- Every integration produces a deployable, uniquely identified build traceable to its source revision.
- No manual version decision is needed between releases.
- Pre-release versions sort below the release they follow, which can confuse tooling or people that compare versions; they must not be published to registries where precedence matters.
- A missing back-merge or a shallow checkout silently produces outdated pre-release versions such as `0.0.0-dev.N`.
- The release commit made on `develop` also triggers a pre-release build, which is expected and harmless.
- Short retention keeps storage costs low but makes older development builds unavailable for investigation.
