# Agent Instructions

These instructions apply to the entire repository. Consult relevant ADRs before proposing or changing a decision, and preserve the decision log's history.

## Creating or Updating ADRs

- Before creating an ADR, search existing titles and content for substantially similar decisions.
- If an existing decision only needs clarification or completion, update that ADR. If the direction materially changes, preserve the accepted decision and create a new ADR linked as `Amends` or `Supersedes`, as appropriate. Never silently rewrite the historical record.
- Record one decision per ADR. Explain the context, rationale, chosen decision, and material consequences and trade-offs clearly enough for readers who did not participate in the original discussion.
- Prefer decisions independent of language, framework, platform, or tool where practical. If a decision depends on a specific technology, state that scope explicitly in the ADR.
- Follow the repository's established structure, format, statuses, and metadata. Do not introduce template requirements without a reason.
- Link other ADRs when a real and useful relationship exists. Choose the correct relationship (`Relates to`, `Amends`, or `Supersedes`) and maintain bidirectional links when the tooling supports them. Do not infer a relationship merely because ADRs share tags or terminology.
- Prefer [ADRS by Josh Rotenberg](https://github.com/joshrotenberg/adrs) for managing this repository. Use its MCP server first for searching, creating, updating, linking, and validating ADRs. Use its command-line interface as the last tooling option when MCP is unavailable or cannot perform the required operation. Edit files directly only for unsupported operations or to correct inconsistencies, preserving the established format.
- Known MCP status-update bug: a status change may update the YAML frontmatter without updating the body's `## Status` section. Verify both after every status change and correct the body directly if needed. Keep the README status synchronized; do not rely solely on a successful MCP response.
- After creating or editing an ADR, update its row in the `README.md` index with an accurate status and summary. Add a row for each new ADR.
- Check consistency among frontmatter, the `Status` section, links, filenames and numbers, and the README.
- At the end of every work session that changes ADRs or their repository documentation, run `doctor` and address reported issues before considering the work complete.

## General Repository Work

- Read applicable ADRs before implementing changes that affect recorded decisions. Do not contradict an accepted ADR without explicitly identifying the need to revise or supersede it.
- Preserve pre-existing local changes and keep edits within the requested scope.
- Keep each file's existing language and formatting; ADRs are in English and the README is in Portuguese.
- Do not mark a proposed ADR as accepted without explicit approval.

## References

- [Architectural Decision Records](https://adr.github.io/)
- [ADR Practices](https://adr.github.io/ad-practices/)
