---
number: 11
title: Establish a consistent process for creating and maintaining ADRs
date: 2026-10-24
status: accepted
tags:
- adr
- documentation
- decision-records
- governance
---

# 11. Establish a consistent process for creating and maintaining ADRs

Date: 2026-10-24

## Status

Accepted

## Context

ADRs form a decision log: each record should capture one significant decision and enough rationale for future readers to understand it. Duplicated decisions, implicit technology assumptions, missing relationships, or an outdated index make the log less useful to people and agents. The ADR community site describes ADRs as records of a decision and its rationale, and points to templates and practices for making decisions sustainable: https://adr.github.io/ and https://adr.github.io/ad-practices/.

## Decision

Before creating an ADR, search the existing decision log for a substantially similar decision. If one exists, update it when clarifying or completing the same decision; when the decision materially changes, preserve the accepted record and create a new ADR that explicitly amends or supersedes it. Do not silently rewrite historical decisions.

Keep each ADR focused on one decision. Describe the context and rationale, the chosen approach, and material consequences and trade-offs clearly enough for readers unfamiliar with the original discussion. Prefer language- and framework-neutral decisions when that is practical. When a decision depends on a specific language, framework, platform, or tool, state that scope explicitly.

Use the repository's established template and status conventions. Link related ADRs whenever a meaningful relationship exists, using an appropriate relationship such as Relates to, Amends, or Supersedes and keeping the links bidirectional when the tooling supports it. Do not add links that imply a relationship that does not exist.

Prefer [ADRS by Josh Rotenberg](https://github.com/joshrotenberg/adrs) for managing this decision log. Use its MCP server first for searching, creating, updating, linking, and validating ADRs; use its command-line interface as the last tooling option when MCP is unavailable or cannot perform the required operation. Direct file edits remain a fallback for operations the tooling cannot complete or for correcting inconsistencies, while preserving the established format. This tooling choice applies to maintaining this repository and does not require other ADRs to prescribe a technology.

Known MCP status-update bug: changing an ADR's status through MCP may update the YAML frontmatter without updating the body's `## Status` section. After every status change, verify both representations and correct the body directly if needed. Update the README status as part of the same change; do not rely solely on the MCP success response.

After creating or editing an ADR, update the README index and its summary or status so the repository's overview remains consistent with the records. At the end of every work session that changes ADRs or their repository documentation, run `doctor` and address reported issues before considering the work complete.

## Consequences

Searching before creation reduces duplicate or conflicting records but adds a small review step. Keeping decisions focused, documenting scope and trade-offs, and linking related records improves discoverability and preserves the rationale for future readers. Maintaining the README adds a synchronization responsibility whenever an ADR changes; review the index alongside the ADR update. Running `doctor` at the end catches repository issues before changes are considered complete. New decisions that change accepted guidance require a new linked record rather than erasing the historical decision.
