---
number: 14
title: Use annotated version tags as the single source of truth for release versions
date: 2026-10-29
status: accepted
tags:
- versioning
- git
- tags
- release
links:
- target: 13
  kind: relatesto
- target: 16
  kind: relatesto
- target: 17
  kind: relatesto
- target: 18
  kind: relatesto
---

# 14. Use annotated version tags as the single source of truth for release versions

Date: 2026-10-29

## Status

Accepted

## Context

A release version can be recorded in several places: project files, build configuration, the changelog, and version control tags. When it is recorded in more than one place, the copies drift; a tag may say `0.2.0` while a project file still says `0.1.0`, and nobody can tell which one the shipped artifact really carries.

A tag in version control is bound to one exact source revision. Branches move as new commits arrive, but a tag keeps pointing to the same commit, which makes it a reliable marker of the code a version was built from.

This decision applies to repositories hosted in Git. The tag naming and immutability rules are independent of language, build tool and hosting platform.

## Decision

The version of a release is defined only by an annotated Git tag named `vMAJOR.MINOR.PATCH` (for example `v0.2.0`), following the versioning rules of ADR 13.

### Tags

- Create release tags as **annotated** tags (`git tag -a`), so they record author, date and message. Lightweight tags are not used for releases.
- The `v` prefix is mandatory. Automation recognizes release tags by the `v*.*.*` pattern; tags outside this pattern are not releases.
- Create the tag on the commit of the release line that contains the release changes, after that commit exists in the remote repository, and push the tag explicitly.

### No version stored in project files

Do not store the release version in source files such as project manifests or shared build configuration. The build pipeline derives the version from the tag and injects it into the build. Local builds without an injected version produce the tooling's default version and are not releases.

### Immutability

A published version is immutable. Once a version may have been consumed, its tag is never moved, deleted or recreated; problems are fixed in a new version, usually the next `PATCH`. Recreating a tag is acceptable only when the release has demonstrably not been published or consumed, such as when its release pipeline failed before publishing anything.

## Consequences

- There is exactly one place to look for the version of a build, and it is bound to an exact source revision.
- Project files never need a "bump version" edit, which removes a source of merge conflicts and mismatches.
- Builds need access to tags and enough history to resolve them, which pipelines must configure explicitly.
- Local builds report a default version rather than the release version; reproducing a release build locally requires passing the version manually.
- Mistakes in a published release cannot be corrected by retagging; they cost a new version number.
- Pushing a branch does not push tags, so forgetting to push the tag is a common failure that blocks the release.
