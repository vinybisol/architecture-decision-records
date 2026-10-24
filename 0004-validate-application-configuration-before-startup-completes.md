---
number: 4
title: Validate application configuration before startup completes
date: 2026-10-22
status: accepted
tags:
- configuration
- validation
links:
- target: 10
  kind: relatesto
---

# 4. Validate application configuration before startup completes

Date: 2026-10-22

## Status

Accepted

## Context

Missing or invalid configuration can make deployments appear healthy until an affected feature is used. Applications need an explicit, testable configuration contract independent of programming language, framework and configuration source.

Configuration validity and external dependency availability are different concerns. A valid connection setting does not prove that a remote service is reachable; operational readiness is a separate decision.

## Decision

Define a configuration contract for each application-owned capability. It specifies required settings, accepted formats and ranges, relationships between settings, and any deliberate defaults. Settings required only by an enabled capability are validated when that capability is enabled.

Resolve and validate effective configuration before startup completes and before accepting normal traffic. Missing or invalid required configuration must fail startup with an actionable diagnostic identifying the setting or rule, without exposing its sensitive value. Do not silently replace invalid settings with defaults.

Consumers receive validated configuration through a structured representation appropriate to the implementation environment. Keep loading and validation at explicit configuration boundaries instead of scattering reads of raw environment variables or configuration sources throughout business logic. Use native framework facilities where suitable; no particular options API, class model or library is required.

Secrets must not be committed, logged or embedded in browser-delivered configuration. Browser applications validate public configuration before enabling affected features; configuration failures must be visible rather than appearing as a successful initialization.

If runtime configuration changes are supported, validate a candidate before applying it. Reject invalid updates explicitly and retain the last valid configuration where safe. Otherwise, document that changes require a restart or redeployment.

Test missing, invalid and valid configuration, including conditional requirements and redaction of diagnostics. External connectivity checks belong to readiness validation, not configuration parsing.

## Consequences

- Invalid required configuration is detected before normal operation instead of on the first affected request.
- Explicit contracts and controlled configuration boundaries improve testability without requiring a particular language or framework.
- Each new setting needs documented validation and default behavior, but not necessarily a new class or abstraction.
- Successful configuration validation does not guarantee external availability; deployment and runtime readiness checks remain necessary.
- Runtime reload introduces additional validation and consistency responsibilities and is not required by this decision.
