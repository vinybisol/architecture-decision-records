---
number: 17
title: Trigger releases from version tags and build each release artifact once
date: 2026-10-29
status: accepted
tags:
- release
- ci-cd
- build-once-deploy-many
- artifacts
links:
- target: 14
  kind: relatesto
- target: 12
  kind: relatesto
- target: 10
  kind: relatesto
- target: 19
  kind: relatesto
---

# 17. Trigger releases from version tags and build each release artifact once

Date: 2026-10-29

## Status

Accepted

## Context

A release needs a build that corresponds exactly to the released source and to the version it declares. If releases are built on every push to `main` (ADR 15), the version must be inferred from somewhere other than the trigger, and a merge that was not meant to be a release would still produce one.

Rebuilding the same version for each environment is also risky. A later build can use different dependency resolutions, toolchain versions or base images, so the binary approved in one environment is not necessarily the binary deployed to the next.

This decision is independent of language and CI/CD platform.

## Decision

### Trigger

The release pipeline runs only when a release tag matching `v*.*.*` (ADR 14) is pushed. Merging into `main` integrates code but does not build or publish a release. The version is taken from the tag name, without the `v` prefix.

### Build once, deploy many

The release pipeline builds the artifact once, from the tagged revision, and injects the tag's version into it. That same artifact is stored and promoted between environments, such as staging and production. Environments are never served by rebuilding the version. Environment-specific settings are supplied as configuration at deployment time, not compiled into separate builds.

### Publishing

Publishing happens only after the build succeeds; a release is never published for a revision that did not build. The published release:

- carries the tag's name;
- includes the built artifact as a downloadable asset;
- uses as its notes the section of the changelog for that version (ADR 12). If the section is missing, the pipeline fails without publishing.

The tag-triggered pipeline is the single entry point for a release. If the process grows, it is split into reusable stages invoked by that pipeline, not into separate pipelines triggered by release events, which some platforms do not raise for events created by the pipeline's own credentials.

### Release definition must exist at the tag

Because the pipeline is triggered by the tag, the release pipeline definition used is the one present in the tagged revision. Changes to the release pipeline must reach `main` before a tag is created. A failed release whose tag was never consumed may be retagged under the conditions of ADR 14; otherwise a new version is released.

Promotion of the artifact into each environment is subject to the deployment gate of ADR 10.

## Consequences

- Each released version maps to exactly one revision and one artifact, which makes deployments reproducible and auditable.
- What was tested in one environment is byte-for-byte what reaches the next.
- Releasing is an explicit act (pushing a tag), separate from integrating code.
- Artifacts must be retained long enough to be promoted and redeployed, which consumes storage.
- Configuration must be externalized, because nothing environment-specific can be compiled in.
- A fix to the release pipeline only takes effect for tags created after it reaches `main`.
