---
number: 19
title: Expose the build-assigned version at runtime
date: 2026-10-29
status: accepted
tags:
- versioning
- observability
- runtime
- health
links:
- target: 17
  kind: relatesto
- target: 18
  kind: relatesto
- target: 10
  kind: relatesto
---

# 19. Expose the build-assigned version at runtime

Date: 2026-10-29

## Status

Accepted

## Context

Versions are assigned by the pipeline at build time (ADR 14, ADR 17, ADR 18) rather than stored in project files. Once an artifact is deployed, people investigating an issue, support teams and other systems need to know which version, and which source revision, is actually running. Without a runtime answer they have to infer it from deployment logs, which is slow and error-prone when several environments run different builds.

Backend services and browser applications both need this. The mechanism for embedding a value at build time differs per language and framework, but the expected behavior is the same.

## Decision

Every deployable application embeds the version assigned by its build and exposes it at runtime. This applies to backend services and browser applications alike.

### What is exposed

The exposed value is the full version string assigned by the pipeline, including build metadata:

| Build | Exposed version |
| --- | --- |
| Release | `0.2.0+e4f5a6b` |
| Pre-release | `0.2.0-dev.15+a1b2c3d` |
| Local, without an injected version | The tooling's default, or a fixed marker such as `0.0.0-local` |

The value is read from what the build embedded, never from a separately maintained file, environment variable set at deployment, or a constant edited by hand. If the build-assigned version cannot be determined at runtime, the application reports a clear placeholder such as `unknown` rather than failing.

### Where it is exposed

- **Backend services** expose the version in the response of their health or status endpoint and include it in structured logs and telemetry as a resource attribute.
- **Browser applications** make the version available in the running application, at least through a non-intrusive place in the interface (such as an "about" area or footer) or a console/diagnostic entry, and include it in error reports and telemetry they send.

Exposing the version and short revision is permitted public information; any other build or environment details follow the minimal-exposure rules of ADR 10.

### Framework mechanisms

How the value is embedded is a per-framework implementation choice, not part of this decision. Examples:

- .NET: pass the version through MSBuild properties (`Version`, `InformationalVersion`) and read `AssemblyInformationalVersionAttribute` at runtime. Avoid duplicating the revision suffix that the SDK may append automatically.
- Angular: generate or replace a build-time constant or environment file with the injected version during the pipeline build, and read it from application code.

Each project documents its mechanism in its own repository.

## Consequences

- Anyone can determine exactly which build is running in any environment, which shortens incident investigation and support.
- Logs, telemetry and error reports can be correlated with a specific release and source revision.
- Every framework needs a small build-time integration, which must be maintained in each project's pipeline.
- Local builds show a default or placeholder version, which must not be mistaken for a release.
- Exposing the version publicly reveals which release is running; this is accepted, while more sensitive build or environment details remain protected.
