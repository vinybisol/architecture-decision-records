---
number: 2
title: Use the Result pattern instead of exceptions for expected failures
date: 2026-10-22
status: accepted
tags:
- error-handling
- result-pattern
links:
- target: 5
  kind: relatesto
---

# 2. Use the Result pattern instead of exceptions for expected failures

Date: 2026-10-22

## Status

Accepted

## Context

Operations can fail for expected business reasons. These outcomes should be explicit and testable without using exceptions as normal control flow or coupling business behavior to a transport or framework.

## Decision

Use Result to represent an operation's success or expected failure as two mutually exclusive alternatives:

```text
Result<Value> = Success(value) | Failure(error)
ResultWithoutValue = Success | Failure(error)

Error = { code, message }
```

- A result is either Success or Failure, never both or neither.
- A value belongs only to Success; an error belongs only to Failure. Success without a value is valid.
- Callers distinguish the alternatives before using their contents.
- An error carries a stable, machine-readable code and a human-readable message. Add structured details only when the operation requires them.
- Return expected failures explicitly. Unexpected failures use the platform's exception mechanism.
- Translate an infrastructure condition into an expected failure only when its meaning for the operation is recognized. Do not indiscriminately catch infrastructure exceptions and convert them into business failures.

Use the language's native alternatives or an equivalent representation that preserves these invariants. This decision does not prescribe inheritance, implicit conversions or helper methods.

Result is independent of transport. HTTP status codes, response serialization and error response formats are separate decisions.

## Consequences

- Expected failure becomes an explicit outcome that can be tested without relying on exception handling.
- Two exclusive alternatives avoid contradictory combinations of success flags, values and errors.
- Callers must handle both alternatives; enforcement depends on the implementation language.
- A small Result model avoids unnecessary error hierarchies and helpers while allowing details where needed.
- Transport adapters remain responsible for translating outcomes into their own contracts.
