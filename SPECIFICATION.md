# Open Agent SLA Specification (OASS) — Draft v0.1

**Status:** Draft, request for comments.
**Author:** Attestum (`https://attestum.ai`).
**License:** CC-BY 4.0. This spec is intentionally free for anyone to implement.
**Discussion:** `github.com/attestum-ai/agent-sla-spec` (issues & pull requests welcome).

---

## 1. Purpose

The Open Agent SLA Specification (OASS) defines a vendor-neutral schema for expressing **service-level agreements for production AI agent tasks**. It is designed so that:

- An enterprise can write one SLA per agent task and have it interpreted the same way by any compliant orchestrator, router, observability tool, or audit pipeline.
- A regulator or auditor can read an OASS record and tie it to specific obligations (EU AI Act Article 15/17/21, SOC 2 Common Criteria 7 & 8, SR-11-7, MAS FEAT).
- A control plane can automatically enforce the SLA, including automatic rollback, and produce a provably complete audit trail.

This is a spec, not a product. The spec is free. A reference implementation is in development at `github.com/attestum-ai/attestum-engine`.

## 2. Scope and Non-Goals

**In scope:**
- Defining the structure of an Agent SLA Contract.
- Defining the minimum audit record emitted per agent run.
- Defining the rollback decision procedure.
- Mapping OASS fields to the regulatory frameworks listed above.

**Non-goals:**
- Specifying how an agent is built (framework-neutral).
- Specifying which models can be used (provider-neutral).
- Defining evaluation methodologies (the contract names the metric; the methodology is attached as a referenced artifact).
- Prescribing pricing or commercial terms.

## 3. Terminology

- **Agent Task** — a named unit of agent work with a defined input shape, success criterion, and blast-radius bound. Example: "Tier-1 customer-support response for the Shopify integration queue."
- **Agent Run** — one execution of an Agent Task, producing a trace of LLM calls, tool calls, retrieval steps, and a final output.
- **Signed SLA** — a cryptographically-hashed agreement between the agent operator and the control plane, defining the SLA Contract (Section 4) for a specific Agent Task at a specific point in time.
- **Control Plane** — the system that enforces the SLA, logs the audit record, and executes rollback. Attestum is one implementation; others MAY exist.
- **Incumbent Path** — the pre-existing agent orchestration that the new routing policy is being compared against, if any.
- **Rollback** — automatic reversion of an Agent Task to its Incumbent Path upon an SLA breach.

## 4. SLA Contract Schema

An SLA Contract is a signed JSON document. The canonical form:

```json
{
  "oass_version": "0.1",
  "contract_id": "sha256-of-contract-body",
  "task": {
    "name": "string, stable identifier",
    "category": "customer_response | document_processing | browser_tool | other",
    "description": "string, human-readable"
  },
  "parties": {
    "operator": { "name": "string", "signer": "string", "signed_at": "ISO-8601" },
    "control_plane": { "name": "string", "signer": "string", "signed_at": "ISO-8601" }
  },
  "effective_from": "ISO-8601",
  "effective_until": "ISO-8601",
  "primary_metric": {
    "name": "string",
    "definition_url": "string (IPFS, git, or https with content hash)",
    "threshold": { "operator": ">= | <= | == | !=", "value": "number", "unit": "string" },
    "rolling_window": { "runs": "integer", "seconds": "integer" },
    "rollback_condition": {
      "type": "consecutive_windows | percent_below_threshold",
      "value": "number"
    }
  },
  "guardrails": [
    {
      "name": "string",
      "definition_url": "string",
      "failure_is_rollback": "boolean",
      "immediate": "boolean (true = rollback on single run failure)"
    }
  ],
  "latency": {
    "end_to_end_p99_ms": "integer",
    "control_plane_p99_ms": "integer (SHOULD be <= 10)"
  },
  "test_set": {
    "hash": "sha256",
    "size": "integer",
    "stratification": "string, describes how the test set was drawn"
  },
  "incumbent_path": {
    "description": "string",
    "baseline_metric_value": "number | null"
  },
  "regulatory_mapping": {
    "eu_ai_act_articles": ["15", "17", "21"],
    "soc2_controls": ["CC7.1", "CC7.2", "CC8.1"],
    "other_frameworks": ["SR-11-7", "MAS-FEAT", ...]
  },
  "signatures": [
    { "signer": "string", "algorithm": "ed25519", "signature": "base64" }
  ]
}
```

### 4.1 Required Fields

An OASS-compliant SLA Contract MUST include:

- `oass_version`
- `contract_id` (SHA-256 of the canonical JSON excluding the `signatures` block)
- `task.name`, `task.category`
- `parties.operator`, `parties.control_plane`
- `effective_from`
- `primary_metric` with all sub-fields
- At least one entry in `guardrails`
- `latency.control_plane_p99_ms`
- `test_set.hash` (if the primary metric is evaluated against a held-out set)
- At least one entry in `signatures`

### 4.2 Task Categories

Three categories are specified in v0.1. Implementations MAY add new categories with a `x-` prefix for experimentation; standardization of additional categories is done via spec amendment.

- **`customer_response`** — agent takes a user query and produces a direct or delegated response. Typical guardrails: PII leak, prohibited topic, tool-call allowlist, runaway loop, latency.
- **`document_processing`** — agent ingests documents and emits structured output. Typical guardrails: schema validity, critical-field presence, hallucination check, citation fidelity, latency.
- **`browser_tool`** — agent takes multi-step tool-using actions in external systems. Typical guardrails: unauthorized domain, state-changing action outside allowlist, credential leak, runaway loop, cost overrun, latency.

### 4.3 Signatures

An SLA Contract MUST be signed by at least one authorized principal from each party. Ed25519 is the recommended algorithm. Signature covers the contract body excluding the `signatures` field itself. Implementations SHOULD pin the public keys in a persistent, tamper-evident store (IPFS, Sigstore, internal PKI).

## 5. Agent Run Audit Record

For every Agent Run executed under an OASS contract, the control plane MUST emit an audit record conforming to:

```json
{
  "oass_version": "0.1",
  "trace_id": "uuid",
  "contract_id": "sha256-of-contract-in-force",
  "task_name": "string",
  "task_version": "sha256-of-agent-definition",
  "tenant_id": "string",
  "timestamps": {
    "received": "ISO-8601",
    "first_decision": "ISO-8601",
    "final_response": "ISO-8601"
  },
  "step_count": "integer",
  "tool_calls": [
    {
      "name": "string",
      "args_hash": "sha256",
      "latency_ms": "integer",
      "result_status": "success | error | policy_denied"
    }
  ],
  "routing": {
    "incumbent_path": "string",
    "attestum_path": "string",
    "reason": "string"
  },
  "tokens": { "input": "integer", "output": "integer" },
  "cost": { "usd": "number", "baseline_usd": "number | null" },
  "cache_hits": "integer",
  "latency": {
    "control_plane_overhead_ms": "integer",
    "provider_latency_ms": "integer"
  },
  "sla_evaluation": {
    "primary_metric_score": "number | null",
    "guardrail_results": [
      { "name": "string", "passed": "boolean", "detail": "string | null" }
    ],
    "rollback_triggered": "boolean",
    "rollback_reason": "string | null",
    "incumbent_response_served": "boolean"
  },
  "regulatory_evidence": {
    "eu_ai_act_article_15": { "robustness_test_id": "string | null" },
    "eu_ai_act_article_21": { "post_market_event_type": "nominal | anomaly | rollback | incident" },
    "soc2_cc7_change_control_id": "string | null",
    "soc2_cc8_risk_acceptance_id": "string | null"
  },
  "signature": { "algorithm": "ed25519", "signature": "base64", "signer": "string" }
}
```

Records MUST be immutable after signature. Append-only storage (Merkle-linked logs, blockchain-anchored receipts, WORM storage) is RECOMMENDED for records supporting regulated workloads.

## 6. Rollback Decision Procedure

An OASS-compliant control plane MUST evaluate rollback on every Agent Run as follows:

1. For each guardrail where `immediate = true`: if the run fails the guardrail, the Attestum path's response MUST NOT be served to the end user; the Incumbent Path's response is served instead, OR the user is served a deterministic error response per the contract. `rollback_triggered = true`.
2. For the primary metric: maintain a rolling window per the `rolling_window` specification. If the window violates the `rollback_condition`, all subsequent runs of this task are served via the Incumbent Path until (a) a human operator signs a re-arm event, AND (b) a replay against a fresh sample passes the threshold.
3. Control-plane overhead violation (`control_plane_p99_ms` exceeded for 5 consecutive minutes) MUST trigger rollback regardless of metric results. Our overhead violation is nobody else's problem.

The re-arm event MUST be logged as an audit record with `post_market_event_type = "resolved"` and signed by an authorized principal.

## 7. Regulatory Mapping

OASS v0.1 provides a reference mapping. Implementations SHOULD declare the mapping they use; auditors SHOULD check for full coverage of the declared frameworks.

### 7.1 EU AI Act (Regulation (EU) 2024/1689)

| Article | OASS coverage |
|---|---|
| **Art. 9** (Risk management system) | `regulatory_evidence.eu_ai_act_article_15.robustness_test_id`, linked to the signed test-set hash in the SLA Contract. |
| **Art. 10** (Data governance) | `test_set.stratification`, contract-level; plus tenant-level data-residency attributes (implementation-specific). |
| **Art. 13** (Transparency to deployers) | The OASS contract itself, published to the deployer. |
| **Art. 14** (Human oversight) | `rollback_triggered` records + the re-arm event requirement (Section 6). |
| **Art. 15** (Accuracy, robustness, cybersecurity) | `sla_evaluation.primary_metric_score`, `guardrail_results`, `regulatory_evidence.eu_ai_act_article_15`. |
| **Art. 17** (Quality management system) | `contract_id`, `task_version`, and `regulatory_evidence.soc2_cc7_change_control_id` together form the QMS record trail. |
| **Art. 21** (Post-market monitoring) | `regulatory_evidence.eu_ai_act_article_21.post_market_event_type` per run; aggregated monthly. |
| **Art. 72** (Post-market monitoring plan) | Implementations MUST be able to produce an aggregated post-market monitoring report; the audit record schema is sufficient input. |

### 7.2 SOC 2 Common Criteria

| Control | OASS coverage |
|---|---|
| **CC7.1** (Change management) | `regulatory_evidence.soc2_cc7_change_control_id` + `task_version` hash chain. |
| **CC7.2** (Monitoring) | Audit records are the monitoring evidence. |
| **CC7.3** (Evaluation of security events) | Rollback events with `post_market_event_type = "incident"` and `"anomaly"`. |
| **CC8.1** (Risk mitigation) | Guardrail results + `risk_acceptance_id` for accepted residual risks. |

### 7.3 Other frameworks (non-normative)

- **SR-11-7** (US Federal Reserve, banks): model validation + ongoing monitoring. OASS contracts satisfy the validation documentation requirement; audit records satisfy ongoing monitoring.
- **MAS FEAT** (Singapore): Fairness, Ethics, Accountability, Transparency. OASS's signed SLAs + audit trail + rollback satisfy accountability and transparency requirements.
- **UK AI regulation (sector-specific)**: TBD.
- **NIST AI RMF**: TBD.

## 8. Security Considerations

- Signatures on contracts and audit records MUST use cryptographically strong algorithms (Ed25519 preferred; RSA-3072+ acceptable).
- Audit records SHOULD be chained via Merkle roots or similar to make retroactive tampering detectable.
- Test-set hashes should reference content-addressed storage so the auditor can retrieve the exact bytes.
- Rollback events are themselves audit records and MUST NOT be deletable without a counter-signed retention-policy event.

## 9. Privacy Considerations

- Audit records MUST NOT contain raw user inputs or outputs by default. Inputs/outputs are stored separately, encrypted, and referenced by hash in the audit record.
- Tenants with stricter residency requirements (EU health data, HIPAA PHI) MUST be able to store raw traces in region-restricted storage while audit records flow through a central control plane.

## 10. Versioning and Evolution

- OASS follows semantic versioning. v0.1 is this draft.
- Breaking changes require a new MAJOR version; fields may be added under MINOR.
- Implementations MUST declare the spec version they implement in every contract and every audit record.

## 11. Reference Implementation

A reference implementation of the control-plane behavior described here is in development at `github.com/attestum-ai/attestum-engine` and will be released under an OSI-approved license. The reference implementation is not normative; other implementations are welcome and encouraged.

## 12. Acknowledgements

The structure draws on precedent set by: OpenTelemetry (for the audit-record philosophy), OCI image signatures (Sigstore / Cosign), and Merkle-anchored audit logs from the transparency-log community (Certificate Transparency, Sigsum).

## 13. Open Questions for v0.2

Community input welcome via GitHub issues.

1. Should we add a `slo_budget` field (error budget per reporting period) to support Google-SRE-style SLO management on top of the SLA?
2. Should the `task.category` be an open vocabulary instead of the three standardized values, with a separate registry of category definitions?
3. Should audit records support streaming / incremental emission for long-running agents (hours-long runs)?
4. Should we define a standard "red team" attestation format so third-party evaluations can attach to a contract?
5. Should the spec mandate a specific append-only-log format (e.g., Sigsum, Rekor) or remain storage-agnostic?
