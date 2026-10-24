---
number: 5
title: Use repositories and Unit of Work for persistence
date: 2026-10-22
status: accepted
tags:
- persistence
- repository
- unit-of-work
links:
- target: 1
  kind: relatesto
- target: 2
  kind: relatesto
---

# 5. Use repositories and Unit of Work for persistence

Date: 2026-10-22

## Status

Accepted

## Context

Use cases need to obtain and persist business state without depending on a database library or storage implementation. When an operation changes related data, its persistence boundary must be explicit rather than spread across independently committed repository calls.

Repositories isolate persistence access, while Unit of Work coordinates the changes belonging to a local persistence operation. These responsibilities should remain small and independent of programming language or framework.

## Decision

Define persistence contracts in Application and implement them in Data, following the dependency boundaries in ADR 1.

- Repositories expose intention-revealing operations for obtaining and changing business state. Introduce them around meaningful business boundaries, not automatically for every storage table.
- Repository contracts return domain objects or application data models without exposing storage-library types, query providers or connection details.
- Repositories do not independently commit changes. Repositories participating in the same Unit of Work share its persistence context and local transaction boundary.
- Application coordinates a use case and explicitly asks Unit of Work to commit its related changes together. Read-only operations do not require a commit.
- A successful commit confirms all changes in that local unit; a failed commit must not leave a partially committed unit. Use the storage technology's transaction facilities where necessary to provide this guarantee.
- Surface persistence failures explicitly. Translate only recognized expected conditions into application failures, as described in ADR 2; do not swallow unexpected failures or report an unsuccessful commit as success.

Language-neutral pseudocode:

```text
USE CASE:
    state = repository.load(input)
    apply domain behavior(state, input)
    repository.stage changes(state)
    unit of work.commit()
    return success
```

The pseudocode illustrates coordination, not a required method naming convention or a requirement to write data in every use case.

This decision does not define idempotency, retries, conflict resolution or distributed transactions. Repository and Unit of Work do not automatically prevent concurrent-update conflicts or duplicate effects, and a local transaction does not include calls to external services. Define additional policies when a concrete use case requires those guarantees.

## Consequences

- Use cases can be tested through controlled repository and Unit of Work implementations without a live database.
- Local commit boundaries are visible and coordinated rather than hidden inside repository calls.
- Persistence integration tests must verify commit success and atomic failure; test doubles alone do not establish transaction guarantees.
- Contracts add maintenance overhead and may overlap with facilities already provided by a storage library. Keep adapters thin and reuse native transaction mechanisms.
- Storage-specific query composition remains inside Data, so new access needs may require new repository operations.
- More advanced consistency mechanisms are not introduced speculatively, and this decision does not claim to provide their guarantees.
