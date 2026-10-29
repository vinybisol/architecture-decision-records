---
number: 13
title: Version releases with Semantic Versioning derived from changelog categories
date: 2026-10-29
status: accepted
tags:
- versioning
- semver
- release
links:
- target: 12
  kind: relatesto
- target: 14
  kind: relatesto
- target: 18
  kind: relatesto
- target: 15
  kind: relatesto
---

# 13. Version releases with Semantic Versioning derived from changelog categories

Date: 2026-10-29

## Status

Accepted

## Context

Releases need a version number that tells consumers what to expect from an upgrade: whether it can break them, adds capabilities, or only fixes problems. Without a shared rule, numbers are chosen inconsistently and stop carrying meaning.

The human-written changelog (ADR 12) already classifies every consumer-observable change under Keep a Changelog categories before a release. That classification contains the information needed to choose the next version, so a separate judgment, or a tool that infers the increment from commit messages, would duplicate it and could contradict it.

## Decision

Version every released artifact with [Semantic Versioning 2.0.0](https://semver.org/spec/v2.0.0.html) in the form `MAJOR.MINOR.PATCH`.

| Part | Incremented when | Example |
| --- | --- | --- |
| `MAJOR` | A change breaks compatibility: a public contract is changed incompatibly or removed. | `1.4.2` → `2.0.0` |
| `MINOR` | A capability is added in a backward-compatible way. | `1.4.2` → `1.5.0` |
| `PATCH` | A backward-compatible fix is made. | `1.4.2` → `1.4.3` |

When a part is incremented, the parts to its right reset to zero.

### Initial development

While the version is `0.y.z`, the project is in initial development and its public contract may change at any time. Version `1.0.0` marks the first stable public contract. From then on, incompatible changes require a `MAJOR` increment.

### Choosing the increment

A person chooses the next version at release time from the categories present under `[Unreleased]` in the changelog:

| Content of `[Unreleased]` | Increment |
| --- | --- |
| `Removed`, or a `Changed` entry that breaks a contract | `MAJOR` (during `0.y.z`, `MINOR` is acceptable) |
| `Added` or `Deprecated` | `MINOR` |
| Only `Fixed` and/or `Security` | `PATCH` |

The largest applicable increment wins: `Added` together with `Fixed` is a `MINOR` release. If `[Unreleased]` is empty, there is nothing to release. Commit messages do not determine the version.

### Pre-release and build metadata

Pre-release versions use a hyphenated suffix (`0.2.0-dev.15`) and are considered unstable. Build metadata follows a plus sign (`0.2.0-dev.15+a1b2c3d`), is used only for traceability, such as the source revision, and does not affect version precedence.

## Consequences

- Consumers can judge the risk of an upgrade from the version number alone.
- The version follows directly from changelog categories that are already reviewed in pull requests, so no extra classification step or commit convention is needed.
- Correct increments depend on correct changelog categories; a breaking change recorded as a plain `Changed` entry leads to a wrong version. Reviewers must check whether `Changed` entries break contracts.
- Choosing the version remains a manual step at release time.
- During `0.y.z` consumers get weaker guarantees, which must be communicated until `1.0.0`.
