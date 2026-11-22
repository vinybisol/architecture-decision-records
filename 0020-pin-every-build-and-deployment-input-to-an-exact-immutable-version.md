---
number: 20
title: Pin every build and deployment input to an exact immutable version
date: 2026-11-22
status: accepted
tags:
- dependencies
- versioning
- supply-chain
- reproducibility
- ci-cd
- build-once-deploy-many
links:
- target: 17
  kind: relatesto
- target: 14
  kind: relatesto
---

# 20. Pin every build and deployment input to an exact immutable version

Date: 2026-11-22

## Status

Accepted

## Context

Builds, tests and deployments depend on many inputs besides the source code: third-party packages, language SDKs and runtimes, container base images, CI/CD actions and runner images, and previously built application artifacts. When any of these is referenced by a floating specifier, such as a version range, a wildcard, `latest`, a major-only tag or a mutable container tag, the same revision can resolve to different inputs on different days.

This breaks reproducibility. Build once, deploy many (ADR 17) assumes the artifact promoted between environments is exactly the one that was built and tested, and that a rebuild of an old revision yields the same result. A floating dependency can also change behaviour without a code change, making failures hard to attribute, and it widens the supply-chain attack surface: a tag can be moved to point at malicious content, as happened with compromised GitHub Actions whose version tags were rewritten.

Tags alone are not sufficient for container images and CI/CD actions, because their tags are mutable. Only a content digest or a full commit SHA identifies immutable content.

The decision itself is independent of language and platform. The section *Application to the current stack* states how it applies to the technologies currently in use: .NET and NuGet, Node.js and npm, Docker and GitHub Actions.

## Decision

Every input that influences how an application is built, tested, packaged or deployed is referenced by an exact, immutable version. Floating specifiers are not allowed. This applies equally to release builds, pre-release builds (ADR 18) and pull request validation.

### General rules

- Declare dependencies with an exact version. Do not use ranges, wildcards or aliases such as `*`, `^`, `~`, `>=`, `1.*`, `latest`, `lts` or `stable`.
- Commit the lock file produced by the package manager, and make CI restore in a mode that fails when the lock file is missing or out of date instead of resolving new versions.
- Reference container images and CI/CD actions by an immutable identifier: a content digest (`sha256:...`) for images and a full commit SHA for actions. Keep the human-readable version next to it, in the tag or a trailing comment, so reviewers can see which version is pinned.
- Pin toolchains, SDKs and runtimes to an exact version, both in the repository configuration and in CI/CD, so local and pipeline builds use the same toolchain.
- Do not rely on what happens to be preinstalled on a build agent; install or select the exact version the build needs.

### Application artifacts

Deployments reference the application artifact by its exact release or pre-release version (ADR 14, ADR 18), and container images additionally by digest. The artifact promoted between environments is identified by that version and digest, never by a moving reference such as `latest`, `stable` or a branch name. Convenience tags may be published, but deployments and other pipelines must not consume them.

### Application to the current stack

| Input | Rule |
| --- | --- |
| NuGet packages | Exact versions in package references, no floating versions. Enable `RestorePackagesWithLockFile`, commit `packages.lock.json` and restore with `--locked-mode` in CI. |
| npm packages | Exact versions in `package.json` (`save-exact=true` in `.npmrc`), commit `package-lock.json` and install with `npm ci` in CI. |
| .NET SDK | Exact version in `global.json` with `rollForward` set to `disable`; CI uses the same file, for example `setup-dotnet` with `global-json-file`. |
| Node.js | Exact version in the repository (for example `.nvmrc` or the `engines` field) and the same version in CI. |
| Docker images | `FROM`, Compose files and deployment manifests use `image:tag@sha256:<digest>`. |
| GitHub Actions | `uses: owner/action@<full commit SHA> # vX.Y.Z`; runners use a versioned label such as `ubuntu-24.04`, not `ubuntu-latest`; job and service containers follow the Docker rule. |

### Updating pinned versions

Pinned versions are updated deliberately, through pull requests that run the normal validation pipeline. An automated dependency update tool, such as Dependabot or Renovate, opens those pull requests for packages, SDKs, images and actions, including digest changes. Security updates are prioritised. Updates that affect consumers are recorded in the changelog (ADR 12).

### Enforcement

Where practical, CI rejects floating references, for example through locked-mode restores, linters for Dockerfiles and workflows, or a check that every `uses:` and `FROM` is pinned to a SHA or digest.

## Consequences

- A given revision always resolves to the same dependencies, toolchain and images, so builds are reproducible and the artifact promoted under ADR 17 is exactly what was tested.
- Behaviour changes caused by third-party updates become explicit, reviewable pull requests instead of surprises in unrelated builds.
- Moved or hijacked tags of images and actions no longer affect builds, reducing supply-chain risk.
- Fixes and security patches are no longer picked up automatically; an automated update tool and timely review of its pull requests become required, otherwise dependencies age and accumulate vulnerabilities.
- Update pull requests add review effort and repository noise, which grouping and scheduling in the update tool can mitigate.
- Digests and commit SHAs are opaque, so the readable version must be kept next to them and kept in sync on updates.
- An exact SDK version with roll-forward disabled requires developers and agents to install that version, and SDK patch updates must be applied explicitly.
