---
number: 6
title: Apply database migrations in CD, not at application startup
date: 2026-10-22
status: accepted
tags:
- database
- migrations
- ci-cd
links:
- target: 10
  kind: relatesto
---

# 6. Apply database migrations in CD, not at application startup

Date: 2026-10-22

## Status

Accepted

## Context

Database evolution is a deployment responsibility. Applying migrations during application startup adds runtime complexity, repeats checks on every restart and couples application availability to schema modification.

We need a predictable migration process with isolated environments and an explicit stop condition when migration fails, without prescribing a database engine, framework or hosting platform.

## Decision

Version database schema changes and required data transformations as migrations alongside the application. Do not modify a migration that has been applied to a shared environment; make subsequent changes through new migrations.

### Execution and environment isolation

- Apply migrations through an explicit CD step in every deployed environment, including shared development environments. Application startup and health checks must never apply migrations.
- Local development is the exception: developers execute migrations manually against their local development database when needed, using the same versioned migrations and tooling as CD.
- Each environment has its own database and credentials. Local development must not target a shared or production database by default. Supply secrets through the environment's protected configuration, not source control or logs.
- Separate migration credentials from runtime credentials where supported. Grant only the permissions each role requires; the running application does not need schema-change permissions solely because CD applies migrations.
- Before execution, verify the intended target environment and database without exposing credentials. Serialize migration execution against the same target database to avoid overlapping deployment runs.
- Use reproducible migration tooling associated with the release. Run migrations before promoting or swapping the new application version; promotion also remains subject to readiness checks.

### Compatibility

Migrations applied while the previous application version is running must remain compatible with that version, including if deployment later stops. Split incompatible changes into staged migrations and application releases, or explicitly plan a maintenance window that prevents incompatible application versions from accessing the database.

Review destructive or irreversible changes and define an appropriate recovery approach before execution. Test migrations against both an empty database and a database representing the previously deployed schema.

### Migration failure

If a migration fails, stop the deployment and do not promote or swap the new application version. Record the failed step and safe diagnostic information, and notify the responsible operator through the deployment system.

Stopping deployment does not roll back changes already applied to the database. Transaction support and migration operations determine whether changes can remain partially applied; do not assume the previous application is unaffected.

Before retrying, inspect the migration history and actual database state, determine the cause and choose an explicit recovery action. Depending on the failure, this may mean a safe retry after a transient issue, a corrective migration or restoration from a verified backup. Do not automatically reverse migrations, blindly rerun a partially applied operation or report success after ignoring the failure.

Keep the previous application serving only if it remains compatible and safe. Otherwise, use the operational recovery or maintenance procedure. This decision establishes a stop-and-investigate policy, not an automated rollback mechanism.

## Consequences

- Startup remains focused on initialization and validation, with no schema-change attempts on application restarts.
- CD provides an explicit, reviewable migration step, while developers control when local migrations run.
- Separate databases and credentials reduce accidental cross-environment changes.
- A failed migration blocks release promotion but does not guarantee automatic restoration of the database or uninterrupted service.
- Compatibility planning and a basic recovery procedure are required, especially for destructive changes.
- Manual local execution adds a developer step but avoids a separate migration path embedded in application startup.
