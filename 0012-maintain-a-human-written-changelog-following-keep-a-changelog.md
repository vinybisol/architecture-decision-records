---
number: 12
title: Maintain a human-written changelog following Keep a Changelog
date: 2026-10-29
status: accepted
tags:
- changelog
- release
- documentation
- versioning
links:
- target: 13
  kind: relatesto
- target: 16
  kind: relatesto
- target: 17
  kind: relatesto
---

# 12. Maintain a human-written changelog following Keep a Changelog

Date: 2026-10-29

## Status

Accepted

## Context

Projects need a record of what changes between versions. People who consume a system, such as front-end applications, integrations, and other teams, should be able to learn what was added, changed, removed, or fixed without reading commits or diffs.

There are two common ways to produce this record. One generates it automatically from commit messages, using tools such as Release Please or semantic-release together with Conventional Commits. The other is a changelog written and curated by people, following the [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) convention.

Commit messages are written for developers, not for consumers. They describe implementation details and include refactorings, tests, pipeline changes, and other work with no observable effect for consumers. A changelog generated from them mixes internal noise with relevant changes and uses vocabulary its readers do not understand. Keep a Changelog itself discourages using commit log diffs as a changelog. Automatic generation also makes every commit responsible for classifying its own change correctly; a misclassified commit produces an incorrect changelog entry without anyone reviewing the final text.

Release notes generated from pull request titles have the same problem, because those titles are also written for developers.

The difference is visible when the same set of changes is recorded both ways. A changelog generated from commits could read:

```markdown
### Features
- feat(bills): add ExportService and CsvWriter
- feat: wire export endpoint

### Bug Fixes
- fix: AddMonths edge case in InterestCalculator
- fix(ci): restore cache key
```

The human-written equivalent reads:

```markdown
### Added
- Export bills as CSV.

### Fixed
- Interest for bills due on the last day of the month was calculated one day short.
```

The pipeline fix is irrelevant to consumers, the two export commits describe one capability, and the interest fix only makes sense to readers who know the code.

## Decision

Maintain a `CHANGELOG.md` file at the repository root, written by people and following Keep a Changelog 1.1.0. Commit messages play no role in generating the changelog, and this decision does not require a commit message convention.

### File structure

```markdown
# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added

- Export bills as CSV.

## [0.2.0] - 2026-11-10

### Added

- Filter bills by category.

### Fixed

- Interest for bills due on the last day of the month was calculated one day short.

## [0.1.0] - 2026-10-28

### Added

- List bills for the current month.

[Unreleased]: <repository-url>/compare/v0.2.0...HEAD
[0.2.0]: <repository-url>/compare/v0.1.0...v0.2.0
[0.1.0]: <repository-url>/releases/tag/v0.1.0
```

- `[Unreleased]` is always the first section, even when empty.
- Versions are listed in reverse chronological order.
- Dates use ISO 8601 (`YYYY-MM-DD`) and record the release date.
- Comparison links at the end of the file make each heading navigable: each version compares with the previous one, `[Unreleased]` compares the latest version with the current head, and the first version links to its own release because there is nothing earlier to compare with.
- Entries do not repeat the version number; the section heading already states it.
- Only categories with content are included.

### Categories

| Category | Use |
| --- | --- |
| `Added` | New capabilities. |
| `Changed` | Changes to existing behavior. |
| `Deprecated` | Capabilities that will be removed in a future version. |
| `Removed` | Capabilities removed in this version. |
| `Fixed` | Bug fixes. |
| `Security` | Vulnerability fixes. |

In a project's first version every entry is `Added`, because nothing existed before to change or fix.

### What is recorded

Include an entry only when a consumer can observe the difference.

| Change | Recorded | Category |
| --- | --- | --- |
| New endpoint or capability | Yes | `Added` |
| New field in a response | Yes | `Added` |
| Different behavior for an existing capability | Yes | `Changed` |
| Bug consumers could notice | Yes | `Fixed` |
| Removed endpoint or field | Yes | `Removed` |
| Noticeable performance change | Yes | `Changed` |
| Refactoring that also fixed a bug | Yes | `Fixed` |
| Dependency update that resolves a vulnerability | Yes | `Security` |
| New minimum runtime version required to run | Yes | `Changed` |
| New or changed tests | No | |
| Refactoring without behavior change | No | |
| Pipeline or build adjustments | No | |
| Routine dependency update | No | |
| Changes to the changelog itself | No | |

### How entries are written

Describe the effect on consumers, not the implementation.

| Avoid | Prefer |
| --- | --- |
| Refactor InterestService | Interest for bills due on the last day of the month was calculated one day short. |
| Add BillsController.Export | Export bills as CSV. |
| Change DTO | `dueDate` is now returned in UTC. |
| v0.2.0 - Create filter | Filter bills by category. |

### Workflow for each change

The pull request that introduces a change also adds its entry under `[Unreleased]`. The changelog entry is reviewed together with the code.

Before the change:

```markdown
## [Unreleased]

## [0.2.0] - 2026-11-10
```

After the pull request that adds CSV export:

```markdown
## [Unreleased]

### Added

- Export bills as CSV.

## [0.2.0] - 2026-11-10
```

A pull request with no consumer-observable effect does not change the changelog. If only such changes accumulate since the last release, `[Unreleased]` remains empty and there is nothing to release from the consumer's perspective; those changes ship with the next release that has observable changes.

### Workflow at release time

1. Review `[Unreleased]`: correct wording, merge duplicate entries, and confirm categories.
2. Choose the next version from the categories present (see ADR 13).
3. Rename `[Unreleased]` to the version and release date.
4. Add a new empty `[Unreleased]` section above it.
5. Update the comparison links.
6. Commit the result as part of the release.

Before releasing `0.3.0`:

```markdown
## [Unreleased]

### Added

- Export bills as CSV.

## [0.2.0] - 2026-11-10

...

[Unreleased]: <repository-url>/compare/v0.2.0...HEAD
[0.2.0]: <repository-url>/compare/v0.1.0...v0.2.0
```

After:

```markdown
## [Unreleased]

## [0.3.0] - 2026-11-20

### Added

- Export bills as CSV.

## [0.2.0] - 2026-11-10

...

[Unreleased]: <repository-url>/compare/v0.3.0...HEAD
[0.3.0]: <repository-url>/compare/v0.2.0...v0.3.0
[0.2.0]: <repository-url>/compare/v0.1.0...v0.2.0
```

Release notes published elsewhere, such as a hosting platform's release page, are derived from the released version's section rather than written separately or generated from commits. A release process that cannot find the section for the version being released should fail instead of publishing empty notes.

### Urgent fixes released outside the regular flow

A fix released directly from the released line, without the pending `[Unreleased]` content, creates its own version section directly:

```markdown
## [Unreleased]

## [0.3.1] - 2026-11-22

### Fixed

- Bills with no due date could not be listed.

## [0.3.0] - 2026-11-20
```

When the fix is merged back into the line where `[Unreleased]` accumulates, conflict resolution keeps the pending `[Unreleased]` entries and the new version section below them.

## Consequences

- Consumers read a changelog written in product vocabulary, without internal noise.
- Changelog text is reviewed in the pull request alongside the code that motivates it.
- Commit messages remain free to serve the technical history.
- The release process does not depend on a third-party changelog generator, and each step remains visible and understandable.
- The categories present under `[Unreleased]` make it straightforward to decide the next version increment.
- Keeping the changelog current depends on discipline; a pull request can easily omit its entry. An automated check that fails when a pull request does not change `CHANGELOG.md`, with an explicit opt-out for changes without consumer impact, mitigates this.
- Deciding whether a change is observable to consumers requires judgment, and borderline cases will occur.
- Concurrent pull requests that edit `[Unreleased]` and merges between long-lived branches can produce conflicts in `CHANGELOG.md`.
