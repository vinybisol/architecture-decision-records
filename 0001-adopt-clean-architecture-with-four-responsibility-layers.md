---
number: 1
title: Adopt Clean Architecture with four responsibility layers
date: 2026-10-24
status: accepted
tags:
- architecture
- clean-architecture
- layering
links:
- target: 5
  kind: relatesto
---

# 1. Adopt Clean Architecture with four responsibility layers

Date: 2026-10-24

## Status

Accepted

## Context

When delivery concerns, business rules and persistence are implemented together, changes spread across responsibilities and business behavior becomes difficult to test without external dependencies. As a system evolves, inconsistent feature structures also increase the cost of understanding and maintaining it.

We need explicit responsibility boundaries and controlled dependencies so business rules remain independent of transport, persistence and framework details. The primary goal is maintainability and isolated testing, not a promise that replacing infrastructure will be inexpensive.

This decision applies to the creation of new systems and the incremental adaptation of existing systems, within a small-to-medium system scope. Size alone is not a measure of suitability: domain complexity, coupling between modules and team coordination must also be considered. Larger systems are outside this decision's adoption scope; growth should trigger an architectural review rather than an automatic rewrite.

A single undifferentiated module has lower initial overhead but provides weaker responsibility boundaries. Layering is chosen here to make those boundaries explicit, accepting additional structure and coordination.

## Decision

Organize the system into four separately identifiable modules or projects, with dependencies directed toward business policy:

- Domain: entities, value objects, business invariants and pure calculations. Has no dependencies on the other layers, transport, persistence or application frameworks.
- Application: use cases, orchestration, input/output models and abstractions for external capabilities required by those use cases. Depends on Domain, not on concrete transport or infrastructure implementations.
- Data: persistence adapters, storage mappings and schema evolution. Implements the persistence abstractions required by Application and may depend on Application and Domain. Business rules remain outside this layer.
- Api: transport adapters, request/response contracts and translation between transport semantics and application outcomes. Delegates use cases to Application and does not access persistence directly or implement business rules.

The composition root assembles the system and connects abstractions to concrete implementations. It may reference all layers solely for assembly and configuration. If hosted inside Api, this exception must remain isolated from request handlers and use-case execution. Other infrastructure adapters must respect the same inward dependency rule; the Data module does not justify placing business rules in infrastructure.

Features follow a responsibility rule, not a mandatory file or interface template. Create only the elements a use case needs; a feature does not have to add something to every layer. Introduce abstractions at meaningful dependency boundaries, not automatically for every implementation.

For existing systems, adapt boundaries incrementally while preserving observable behavior and existing contracts unless a change is explicitly agreed. Introduce tests around affected behavior and move responsibilities in bounded steps; adopting this architecture does not require a complete rewrite.

Language-neutral pseudocode:

```text
TRANSPORT ADAPTER handles request:
    input = translate request into use-case input
    outcome = execute use case(input)
    return translate outcome into transport response

USE CASE executes input:
    state = obtain required state through an application-defined port
    outcome = apply domain behavior(state, input)
    if outcome requires persistence:
        persist changes through an application-defined port
    return application outcome

DOMAIN BEHAVIOR:
    enforce business invariants
    calculate or change business state
    do not access transport or persistence

PERSISTENCE ADAPTER implements an application-defined port:
    translate between domain/application needs and storage operations

COMPOSITION ROOT:
    connect ports to adapters
    assemble use cases and transport adapters
```

The pseudocode illustrates responsibility boundaries, not a requirement to perform I/O or persist changes in every use case, nor a specification of transaction semantics.

Create automated architectural tests and run them in continuous integration. They must verify that:

- Domain has no dependencies on the other layers or external delivery/persistence frameworks.
- Application does not depend on Api, Data or concrete infrastructure frameworks.
- Data does not depend on Api.
- Transport handlers do not access Data directly; any Api-to-Data dependency is restricted to the composition root.
- The composition-root exception does not become a path for business logic or persistence access during request handling.

Use dependency checks and boundary-focused tests appropriate to the implementation environment. Document any aspect that cannot be checked automatically and review it explicitly; dependency checks alone do not prove correct responsibility placement.

For incremental adoption, record existing dependency violations as explicit, temporary exceptions with a remediation plan. Architectural tests must prevent new violations while those exceptions are reduced.

## Consequences

- Business rules can be tested without a database, network or transport pipeline; use cases can be tested through controlled implementations of their external ports.
- Changes to infrastructure are less likely to spread into business policy, but replacing infrastructure still requires adapter work, validation and possibly changes to assumptions.
- Explicit responsibility boundaries make navigation and reviews more predictable without forcing every feature into an identical set of files or interfaces.
- Separate modules, contracts and mappings add overhead. Avoid pass-through abstractions and unnecessary elements when they do not protect a meaningful boundary.
- Automated architectural tests make dependency violations visible in continuous integration. They require maintenance and complement, rather than replace, behavioral tests and design review.
- The composition root is an intentional dependency exception and must remain narrow and verifiable.
- Existing systems may temporarily contain old and new structures during incremental adoption. Explicit exceptions and behavior-preserving steps limit disruption without requiring an immediate rewrite.
- This architecture is the default within the small-to-medium system scope. Reassess it when growing domain complexity, cross-module coupling or coordination costs make the boundaries ineffective; system size alone neither proves failure nor requires migration.
