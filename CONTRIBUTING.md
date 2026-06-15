# Contributing to OASS

The Open Agent SLA Specification is maintained as a community-reviewed standard. All contributions welcome.

## Ways to contribute

1. **Open an issue** for feedback, objections, or missing cases. Especially valuable:
   - Missing regulatory mappings (jurisdictions or frameworks we haven't covered)
   - Missing audit fields your compliance or internal-audit team would ask for
   - Rollback-procedure edge cases
   - Ambiguities in the schema
2. **Open a pull request** for fixes, clarifications, or v0.2 candidate additions.
3. **Implement OASS** in your control plane, orchestrator, or observability tool — and open a PR adding your implementation to `IMPLEMENTATIONS.md`.
4. **Comment on open RFCs** in the issues list for specific v0.2 proposals.

## What makes a good issue

- Specific and sourced. "Article 15 doesn't cover cybersecurity adequately" is weak; "Article 15(4)'s cybersecurity obligation maps to [specific clause] but OASS has no corresponding field for [specific artifact]" is strong.
- Example-driven. If you have a production agent that OASS can't express, attach a redacted example.
- Scoped. One concern per issue; link related ones in the thread.

## What makes a good pull request

- Touches the minimal surface area to make the change.
- Preserves backwards compatibility under the same MAJOR version.
- Updates examples in the spec if the schema changes.
- Includes a short rationale in the PR description: what problem, what alternative was considered, what tradeoff was chosen.

## Decision process

For v0.1 → v0.2, all changes are decided by the maintainers (currently the Attestum team) based on community discussion. We expect to transition to a multi-organization steering group once OASS has three or more independent implementations.

Breaking changes require a new MAJOR version. Additive changes may ship under MINOR.

## Code of conduct

Be precise. Be direct. Don't be rude. Maintainers reserve the right to lock or delete threads that stop being about the spec.

## Signing

Specification commits are signed by the author. We recommend contributors sign their commits (`git commit -S`) and we may require it for v1.0+.
