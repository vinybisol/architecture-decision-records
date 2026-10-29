---
number: 15
title: Use a simplified Git Flow branching model with develop and main
date: 2026-10-29
status: accepted
tags:
- git
- branching
- release
- hotfix
links:
- target: 16
  kind: relatesto
- target: 13
  kind: relatesto
---

# 15. Use a simplified Git Flow branching model with develop and main

Date: 2026-10-29

## Status

Accepted

## Context

Work in progress, work ready for the next release, and work already released need to be separated. Without that separation, an urgent production fix cannot ship without also shipping unfinished changes, and it is unclear which state of the code corresponds to what is running in production.

Classic Git Flow solves this with several long-lived and short-lived branch types, including a dedicated release branch per version. For small teams the release branch adds ceremony without much benefit, while the remaining structure is still useful.

This decision applies to repositories hosted in Git and does not depend on language or hosting platform.

## Decision

Use a simplified Git Flow with two long-lived branches and two kinds of short-lived branches.

| Branch | Role | Created from | Merged into |
| --- | --- | --- | --- |
| `main` | Released code. Every merge into `main` corresponds to a release. | — | — |
| `develop` | Integration. Everything here is intended for the next release. | `main` (once) | `main` |
| `feature/*` | One change under development. | `develop` | `develop` |
| `hotfix/*` | Urgent fix for released code. | `main` | `main`, then back into `develop` |

### Scope of a release

Everything merged into `develop` goes into the next release; the promotion from `develop` to `main` takes the whole branch. Selecting individual changes with cherry-picks is not allowed, because it duplicates commits and causes recurring conflicts between the long-lived branches.

The decision about what ships is therefore made when merging into `develop`:

- A change that is not ready is not merged yet.
- A change that must be integrated but not yet enabled is guarded by a feature flag.

### Hotfixes

A hotfix is the only flow that starts from `main`. It contains only the fix, is released as a new `PATCH` version (ADR 13), and is merged back into `develop` afterwards so the fix is not lost in the next release.

### Release branches

Dedicated `release/*` branches are not used by default. A project may adopt them when several people keep merging into `develop` while a release is being stabilized; in that case the branch is created from `develop`, receives the release commit, and is merged into `main` and back into `develop`. Adopting them should be recorded as an amendment to this decision.

Merge strategies between these branches are defined in ADR 16.

## Consequences

- `main` always reflects released code, and `develop` always reflects the next release candidate.
- Urgent fixes ship without unfinished work.
- Release scope is controlled at integration time; teams need discipline, or feature flags, to keep unready work out of `develop`.
- Without release branches, `develop` should not receive changes intended for a later version while a release is being prepared.
- Hotfixes require a back-merge into `develop`, which may conflict, typically in the changelog.
- Feature flags add configuration and code paths that must be removed once the feature is released.
