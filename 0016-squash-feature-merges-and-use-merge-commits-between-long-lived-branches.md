---
number: 16
title: Squash feature merges and use merge commits between long-lived branches
date: 2026-10-29
status: accepted
tags:
- git
- merge-strategy
- branching
- release
links:
- target: 15
  kind: relatesto
- target: 14
  kind: relatesto
- target: 12
  kind: relatesto
- target: 18
  kind: relatesto
---

# 16. Squash feature merges and use merge commits between long-lived branches

Date: 2026-10-29

## Status

Accepted

## Context

Pull requests can be integrated by squashing, rebasing or creating a merge commit. The choice affects more than how the history looks.

Squashing and rebasing create new commits with the same content but without a link in the commit graph to the original commits. Between short-lived feature branches and `develop` (ADR 15) this is useful: draft commits disappear and each change becomes one readable commit.

Between the long-lived branches `develop` and `main` it is harmful. Release tags (ADR 14) are created on `main`. If `main` and `develop` are not connected in the graph, tags on `main` are not ancestors of `develop`, so tools that resolve the latest version from history, such as `git describe`, no longer find them. Pull requests between the branches then show commits that were already integrated, along with conflicts that do not reflect any real difference in content.

This decision applies to repositories hosted in Git. Hosting platform names for merge options vary; the underlying Git operation is what matters.

## Decision

Use the following strategy for each kind of pull request:

| Pull request | Strategy |
| --- | --- |
| `feature/*` → `develop` | Squash merge |
| `develop` → `main` | Merge commit |
| `hotfix/*` → `main` | Merge commit |
| `main` → `develop` (back-merge) | Merge commit |

Squashing or rebasing between `develop` and `main`, in either direction, is not allowed.

### Squashed commit message

Because a feature pull request becomes a single commit, its title and description become that commit's message. Write the pull request title as a descriptive, imperative summary of the change; commits inside the feature branch may be informal.

### Mandatory back-merge

After every release or hotfix, merge `main` back into `develop` with a merge commit. The merge commit created on `main`, which the release tag points to, does not otherwise exist in `develop`, and the new tag would remain unreachable from it. The back-merge normally has no content conflicts after a regular release. After a hotfix, conflicts in the changelog are resolved as described in ADR 12.

A tag is reachable from `develop` when `git merge-base --is-ancestor vX.Y.Z develop` succeeds.

### Enforcement

Hosting platforms usually allow merge methods to be restricted only per repository, not per target branch. Where per-branch enforcement is unavailable, all methods stay enabled and this rule is applied in review.

## Consequences

- `develop` keeps a concise history with one commit per feature.
- Release tags stay reachable from `develop`, so pre-release versions are derived correctly (ADR 18) and pull requests between long-lived branches show only real differences.
- A mistaken squash or rebase between `develop` and `main` breaks the graph link; repairing it requires an explicit merge commit between the branches.
- Every release and hotfix adds a back-merge step, which is easy to forget.
- The original commits of a feature branch are not kept in `develop`; their detail survives only in the pull request.
- Where the platform cannot enforce the rule per branch, it depends on reviewer attention.
