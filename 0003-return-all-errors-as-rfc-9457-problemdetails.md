---
number: 3
title: Return all errors as RFC 9457 ProblemDetails
date: 2026-10-22
status: accepted
tags:
- error-handling
- api
- problem-details
---

# 3. Return all errors as RFC 9457 ProblemDetails

Date: 2026-10-22

## Status

Accepted

## Context

HTTP API clients need a predictable error representation for business failures, request validation, framework-generated errors and unexpected application failures. Inconsistent response formats and clients interpreting human-readable messages as identifiers make integrations fragile.

The contract must be independent of programming language and framework. Production responses must disclose only controlled, client-safe information, while restricted development and QA environments may provide additional diagnostics. Client reports must be correlatable with server-side diagnostic records without exposing internal exception details.

## Decision

Use RFC 9457 Problem Details, serialized as application/problem+json, for HTTP error responses controlled by the application when HTTP permits a response body. This includes expected application failures, request validation, authentication/authorization errors, routing errors and unexpected failures. Do not wrap Problem Details in a response envelope.

Each implementation may use its framework's native mechanisms or equivalent adapters, provided they preserve this contract. This decision does not prescribe middleware names, ordering APIs, libraries or programming languages. HTTP status-code selection is a separate policy.

### Minimum contract

Require the following fields as an application-level policy; RFC 9457 itself does not require all of them:

- type: a stable URI reference identifying the problem type. Prefer an absolute URI with documentation for application-specific types. Use about:blank only when HTTP status semantics are sufficient; do not use a class name or stack trace as a type.
- title: a short human-readable summary of the problem type, consistent between occurrences except for localization.
- status: the numeric HTTP status used in the actual response. Do not return HTTP 200 for an error merely because the body contains an error status.
- detail: a controlled, human-readable explanation of this occurrence that helps the client understand or correct the problem. For unexpected failures, use a generic safe explanation, not the raw exception message.
- code: an extension containing a stable, documented, machine-readable application error code, consistent with the problem type. Provide controlled codes for framework-generated and unexpected errors as well.
- traceId: an extension containing an opaque correlation identifier for the server-side handling context. It is not a credential, an authorization mechanism or a guarantee that telemetry was retained.

Clients use type as the primary problem-type identifier and may use documented code values for application-specific handling. Clients must not parse title or detail to determine behavior and must tolerate unknown extension fields. Diagnostic mode must not change the meaning of type, code or status.

instance is optional. When supplied, it identifies this specific occurrence, not merely the shared route, and must not expose sensitive URL components.

For validation failures requiring multiple issues, an optional documented errors extension may contain issue codes, safe messages and input locations where applicable. Do not echo rejected secrets or sensitive input values. Additional structured fields should be introduced only when clients need them.

### Information disclosure by environment

- Production: return only explicitly allowed, client-safe fields and messages. Do not expose stack traces, raw exception messages, internal queries, file paths, credentials, tokens or sensitive personal data.
- Development and QA: additional sanitized diagnostics may be returned in a separate diagnostics extension only when explicitly enabled and access is restricted to authorized users or a trusted local environment. The environment name alone does not authorize disclosure. Diagnostic responses must not be shared-cached.
- Without explicit diagnostic enablement, use the production-safe policy in every environment. Secrets and sensitive personal data must never be included, even in diagnostic mode. Prefer server-side diagnostic records over exposing internals in responses.

### Correlation and observability

Reuse the active tracing context when available. Otherwise generate a server-controlled correlation identifier and include the same identifier in relevant diagnostic records. Client-supplied context must be validated according to the tracing protocol, not blindly echoed as a trusted identifier.

If an observability platform is integrated, the returned traceId must correlate with the context recorded by that platform; do not generate an unrelated response-only identifier. Preserve correlation across downstream calls when distributed tracing is available. Useful records include the application error code, HTTP status and relevant internal exception context, subject to redaction and access controls. Expected business failures need not all be recorded as unexpected exceptions.

Vendor selection, exporters, sampling, retention, dashboards and alerting belong to a separate observability decision. A traceId remains useful even when only correlated logs are available.

### Scope and verification

Test the contract for application and framework-generated errors, unexpected failures, production-safe responses, restricted diagnostic mode and correlation identifiers. Where error handling is extensible, use shared mapping mechanisms to avoid divergent formats.

Errors generated by infrastructure outside application control, responses already started, connection aborts and responses where HTTP forbids a body cannot always follow this representation. Document those boundaries rather than claiming an unconditional guarantee.

## Consequences

- Clients receive a stable error contract across implementation technologies without coupling Result or business rules to HTTP.
- Stable problem types and codes support machine handling; human-readable text can evolve or be localized without becoming the protocol.
- Production-safe responses limit disclosure while controlled diagnostic mode supports investigation in restricted environments.
- Correlation identifiers connect client reports to logs or traces without requiring a particular observability vendor or guaranteeing telemetry retention.
- Framework defaults may need customization to satisfy the minimum contract, and integration tests must cover those adaptations.
- Additional validation or diagnostic fields are optional extensions, not a requirement to build a broad error hierarchy or response envelope.
- A common representation cannot cover failures outside the application's control or situations in which an HTTP response body cannot be produced.
