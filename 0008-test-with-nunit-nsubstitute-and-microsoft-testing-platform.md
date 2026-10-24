---
number: 8
title: Test with NUnit, NSubstitute and Microsoft.Testing.Platform
date: 2026-10-22
status: proposed
tags:
- dotnet
- testing
- nunit
- nsubstitute
- microsoft-testing-platform
---

# 8. Test with NUnit, NSubstitute and Microsoft.Testing.Platform

Date: 2026-10-22

## Status

Proposed

## Context

This decision applies exclusively to .NET projects. It does not prescribe testing tools for other technology stacks.

Every change must be covered by unit tests for business logic and by integration tests for endpoints (including authentication and database access). The suite must stay fast enough for iterative development, run reliably in CI against a real database, and produce coverage without the legacy VSTest stack.

Realistic synthetic data can make test setup easier to read, but generated values must not make the expected outcome unpredictable. Generating data and substituting dependency behavior are separate responsibilities.

## Decision

- **Framework:** NUnit with constraint-model assertions (`Assert.That`) and NUnit.Analyzers; **NSubstitute** for test doubles.
- **Realistic test data:** use **Bogus** to generate synthetic names, emails, addresses, numbers within explicit ranges, and composed object graphs when realistic data is useful. NSubstitute remains responsible for dependency behavior, not data generation. Construct domain entities through their public constructors or factories rather than bypassing invariants.
  - Generate incidental data automatically, but set every value that determines the scenario's expected outcome explicitly. For a minimum-age rule of 18, test ages 17, 18 and 19 deliberately; do not rely on a randomly selected age to cover those boundaries.
  - Keep generated scenarios reproducible using a local seed per generator/test rather than shared mutable global random state. Set date/time reference values explicitly when generation depends on the current time. A seed alone does not guarantee identical output across changed rules or library versions; fix package versions and preserve the relevant inputs when diagnosing a failure.
  - Use synthetic data, not copied production personal data. Realistic-looking data is not a substitute for explicit valid, invalid and boundary cases.
- **Runner:** **Microsoft.Testing.Platform (MTP)**, enabled in `global.json` (`"runner": "Microsoft.Testing.Platform"`), with coverage from `Microsoft.Testing.Extensions.CodeCoverage` (`--coverage` in CI); no `Microsoft.NET.Test.Sdk`/VSTest. Package versions are centralized in `Directory.Packages.props`, and shared test settings live in `tests/Directory.Build.props`.
- **Unit tests:** one project per layer under `tests/UnitTests/` (`Domain`, `Application`, `Api`, `Data`), mirroring `src/`, so only the affected layer needs to run.
- **Integration tests:** a **single** project, `tests/IntegrationTests/Api.IntegrationTests`, using `WebApplicationFactory` against a real PostgreSQL. JWTs are signed locally (`TestTokens`).
  - Each test uses a distinct `firebase_uid` (and therefore `owner_id`), so the global owner filter isolates data without cleaning the database between tests.
  - **Respawn** resets the database **once per fixture** (`[OneTimeSetUp]` in `IntegrationTestBase`), never per test.
  - The assembly is `[assembly: NonParallelizable]`, because parallel fixtures would erase each other's data through Respawn.

## Consequences

- This tool selection is scoped to .NET and does not constrain projects implemented with other stacks.
- Bogus provides readable synthetic data; explicit scenario values and reproducible generation prevent randomness from deciding test outcomes.
- Realistic data generation adds a test dependency and generator rules to maintain, and does not replace domain validation or deliberate edge-case coverage.
- Fast feedback: unit tests run per layer in seconds; integration tests can be scoped to one fixture with `--filter FullyQualifiedName~XEndpointTests`.
- Integration tests exercise the real HTTP pipeline, JWT validation, global query filters and PostgreSQL constraints, which catches issues mocks would hide.
- A lightweight runner and built-in coverage without VSTest.
- Integration tests are slower and require a running PostgreSQL (Docker locally, a service container in CI); they cannot run in parallel.
- With MTP, `dotnet test --filter ...` without `--project` runs across the whole solution and fails with "Zero tests ran" (exit code 8) for projects with no matching tests, so commands must always target a project.
- Isolation depends on discipline: a test that reuses another test's uid would share data.
