# Open Agent SLA Specification (OASS)

**Status:** Draft v0.1 — request for comments.
**License:** [CC BY 4.0](./LICENSE) for the specification text. Reference implementations under the license stated in their own repos.
**Spec document:** [`SPECIFICATION.md`](./SPECIFICATION.md)
**Discussion:** open an issue or pull request in this repo.

---

## What this is

A vendor-neutral schema for expressing **service-level agreements for production AI agent tasks**. Designed so that:

- An enterprise can write one SLA per agent task and have it interpreted the same way by any compliant orchestrator, router, observability tool, or audit pipeline.
- A regulator or auditor can read an OASS record and tie it to specific obligations (EU AI Act Article 15/17/21, SOC 2 Common Criteria, SR-11-7, MAS FEAT).
- A control plane can automatically enforce the SLA — including automatic rollback — and produce a provably complete audit trail.

This is a specification, not a product. The spec is free. A reference implementation is in active development at [attestum-ai/attestum-engine](https://github.com/attestum-ai/attestum-engine).

## What problem this solves

Production AI agents take actions with real consequences: customer replies, tool calls against real APIs, browser interactions with corporate credentials. "We'll look at the logs after something goes wrong" is forensics, not governance.

OASS gives you three things:

1. **A Signed SLA Contract** per agent task — short, structured, hashable, signed by both the operator and the control plane. Defines what "correct" means in numbers, not adjectives.
2. **An immutable Agent Run Audit Record** per run — linking inputs, tool calls, model choice, SLA evaluation, and rollback decisions to the specific contract in force at that moment.
3. **A deterministic Rollback Decision Procedure** — control planes MUST implement this so that governance is enforceable, not advisory.

## Scope at a glance

- **Three initial task categories:** `customer_response`, `document_processing`, `browser_tool`. Other categories welcome via spec amendment.
- **Regulatory mapping:** EU AI Act Articles 9, 10, 13, 14, 15, 17, 21, 72; SOC 2 CC7 and CC8. Non-normative mappings to SR-11-7 and MAS FEAT.
- **Framework-neutral:** works with LangGraph, CrewAI, OpenAI Agents SDK, Anthropic Claude Agent SDK, AutoGen, LlamaIndex Workflows, or any custom orchestrator.
- **Provider-neutral:** OpenAI, Anthropic, Google, Meta, xAI, Mistral, open-weight models served anywhere.

## How to use this repo

- Read [`SPECIFICATION.md`](./SPECIFICATION.md) cover to cover.
- Open issues for **feedback, objections, or missing cases** — especially missing regulatory mappings, missing audit fields your compliance team would ask for, or rollback-procedure edge cases.
- Open pull requests for **fixes, clarifications, or v0.2 candidate additions**.
- If you implement OASS in another control plane, orchestrator, or observability tool, open an issue or PR adding it to the `IMPLEMENTATIONS.md` list (will be created when we have the first one).

## Non-goals

- Specifying how agents are built (framework-neutral).
- Specifying which models can be used (provider-neutral).
- Defining evaluation methodologies (the contract names the metric; the methodology is attached as a referenced artifact).
- Prescribing pricing or commercial terms.

## Versioning

OASS follows semantic versioning. v0.1 is this draft. Breaking changes require a new MAJOR version; fields may be added under MINOR.

## Open questions for v0.2

Community input welcome. See the end of [`SPECIFICATION.md`](./SPECIFICATION.md) for the current list. Top of mind:

1. Should we add an SLO-budget field on top of the SLA for Google-SRE-style error-budget management?
2. Should `task.category` be an open vocabulary with a separate registry, rather than the three values standardized in v0.1?
3. Should audit records support streaming / incremental emission for hours-long agent runs?
4. Should third-party red-team attestations be attachable to contracts in a standard format?
5. Should the spec mandate a specific append-only log substrate (Sigsum, Rekor, Certificate Transparency-style) or remain storage-agnostic?

## Author

Draft authored by the team at [Attestum](https://attestum.ai) — the reliability and compliance control plane for production AI agents. OASS is maintained here as an independent open specification; Attestum is one implementation, others are welcome and encouraged.

## Acknowledgements

Structure draws on precedents from:
- **OpenTelemetry** for the audit-record philosophy.
- **Sigstore / Cosign** for signing and transparency-log integration.
- **Certificate Transparency and Sigsum** for tamper-evident append-only logs.

## License

The specification text is released under [Creative Commons Attribution 4.0 International](./LICENSE). You are free to share and adapt the text, with attribution.
