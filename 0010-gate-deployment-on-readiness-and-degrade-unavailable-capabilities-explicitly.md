---
number: 10
title: Gate deployment on readiness and degrade unavailable capabilities explicitly
date: 2026-10-24
status: proposed
tags:
- reliability
- readiness
- deployment
- graceful-degradation
links:
- target: 4
  kind: relatesto
- target: 6
  kind: relatesto
---

# 10. Gate deployment on readiness and degrade unavailable capabilities explicitly

Date: 2026-10-24

## Status

Proposed

## Context

Valid configuration alone does not establish that an application can use its external dependencies. Failures may arise from connectivity, permissions, incompatible schemas or remote service outages. Deploying an unusable instance can interrupt service, while stopping the entire application for an optional dependency unnecessarily disables unrelated features.

Applications need explicit dependency requirements, bounded operational checks and visible degraded behavior. This applies to backend services and browser applications without prescribing a framework, hosting provider or observability vendor.

## Decision

### Dependency classification

Document external dependencies and classify each as critical to the instance's essential service or optional to that service. Identify the capabilities affected by its failure and the permitted degraded behavior. Optional does not mean errors may be hidden or operations falsely reported as successful. If a dependency cannot be safely checked, document the limitation and an alternative verification strategy.

### Startup and runtime readiness

Validate critical dependencies before declaring an instance ready to accept normal traffic. A process may start to perform checks and expose health information while remaining not ready. Critical dependency failure blocks readiness; optional dependency failure permits explicitly degraded operation where independent capabilities remain safe.

Expose readiness through an endpoint or an equivalent internal command or platform probe. Distinguish readiness from liveness: an external outage is not, by itself, evidence that the process must be restarted. Use bounded, repeatable checks after startup to detect loss and recovery of availability, with policies appropriate to the dependency and without overloading it. Handle failures during real operations as well; a successful probe is not a guarantee that the next request will succeed.

Checks must use timeouts and bounded retries with backoff where appropriate. Prefer lightweight, side-effect-free checks that establish the access actually required, not merely that a host answers. Do not create business data, trigger payments, perform migrations or invoke destructive operations as a health check. Record when a weaker probe cannot prove full capability.

### Deployment gate

Run readiness validation against the candidate deployment in the target environment, using its effective configuration, network path and runtime identity. A successful call from the CD runner alone is insufficient evidence of application connectivity.

Do not promote, swap or route normal production traffic to a candidate that fails critical readiness checks within the configured deployment deadline. Fail the deployment explicitly and retain the previous deployment where the platform permits. Document equivalent promotion gates when swap is unavailable. This gate does not undo schema changes or other deployment side effects; recovery for those changes requires its own plan.

### Browser application behavior

Keep the application shell and independent capabilities usable where possible. When a capability is unavailable, display a clear message in its affected area, prevent unsupported actions and offer a bounded retry or recovery path. Preserve user input where safe and do not show a successful operation when its dependency failed.

Differentiate unavailability from authentication, authorization and validation failures rather than treating every failed request as an outage. Do not require browsers to call internal infrastructure probes or embed infrastructure credentials. Determine capability availability through safe application-facing checks or actual operation outcomes.

### Diagnostics and verification

Expose only the minimum public health information needed; protect detailed dependency diagnostics and redact secrets and sensitive data. Record readiness transitions and failed checks with useful correlation context. Tooling choices and alert policies belong to a separate observability decision.

Test critical failures blocking readiness and promotion, optional failures preserving independent behavior, timeout limits, recovery, and visible browser degradation. Configuration validity is governed separately by ADR 4.

## Consequences

- Deployments are gated by usable critical dependencies rather than configuration validity alone.
- Optional outages do not unnecessarily disable independent capabilities, while affected operations remain explicitly unavailable.
- Separating readiness and liveness avoids restart loops caused solely by external outages.
- Checks add traffic and operational complexity; bounded frequency, timeouts and conservative retry policies are necessary.
- Readiness is a point-in-time signal, not an availability guarantee or a substitute for handling runtime failures.
- Browser degradation and deployment gates require explicit integration and tests rather than relying on framework defaults.
