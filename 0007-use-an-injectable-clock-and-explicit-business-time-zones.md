---
number: 7
title: Use an injectable clock and explicit business time zones
date: 2026-10-22
status: accepted
tags:
- time
- time-zones
- testing
---

# 7. Use an injectable clock and explicit business time zones

Date: 2026-10-22

## Status

Accepted

## Context

Business rules often depend on the current instant or business date. Reading the system clock directly makes these rules difficult to test deterministically, especially around expiration and calendar boundaries.

An instant and a business date are different concepts. UTC identifies an instant consistently, but does not by itself define the user's current month, a local due date or a business closing period.

## Decision

Obtain the current instant through an injectable clock or equivalent replaceable time source. Use native platform facilities where appropriate; no particular language, framework or library is required.

- Application obtains the time needed by a use case. Domain behavior and pure calculations receive explicit instants or dates rather than reading the system clock.
- Capture one reference instant per logical operation when related decisions or timestamps must be consistent. Obtain another reading only when elapsed time or a later event is intentionally being measured.
- Represent event and audit instants unambiguously, normally in UTC. Keep date-only business values as dates rather than converting them arbitrarily into UTC timestamps.
- Define the relevant business time zone explicitly for rules involving local dates, months or closing periods. Derive those values from the reference instant in that zone; do not depend on the server's local time zone. Document handling of ambiguous or nonexistent local times when a rule is affected by clock transitions.
- Expiration rules must define their boundary behavior, including whether equality with the expiration instant is expired.
- Audit timestamps generated in Data must follow the same explicit policy. Prefer the injectable clock for application-owned timestamps. If database-generated timestamps are required, document their authoritative source, UTC representation and any differences from application time; do not assume the two clocks are identical.
- Tests use a fixed clock for deterministic rules and an advanceable clock when the scenario requires time to pass, without relying on real waiting.

Test relevant calendar boundaries, business time zones and exact expiration boundaries. Use automated dependency or prohibited-API checks where practical to detect direct clock reads in business code.

## Consequences

- Time-dependent behavior can be tested deterministically without depending on wall-clock timing or the test machine's time zone.
- Explicit separation of instants and business dates prevents UTC from becoming an accidental business-calendar policy.
- Consistent reference instants avoid contradictory decisions within an operation that crosses a time boundary.
- Consumers that need current time receive an additional dependency or explicit time parameter.
- Database-generated times require documented coordination and integration tests rather than an assumption of a single shared clock.
