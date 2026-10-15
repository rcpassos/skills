---
name: docs-review-rcp
description: Read-only documentation audit for contradictions, requirement coverage, decision status, technical feasibility, and fidelity of Word documents and diagrams.
disable-model-invocation: true
---

Audit the documentation the user identifies and deliver an evidence-backed report in the conversation. Default to Portuguese from Portugal unless the user specifies another language. Complete the review through the final report; a plan alone is not the deliverable.

Stay read-only: preserve documents, generated repository files, and Git state; make no commits. Keep extraction, rendering, and working notes in a temporary directory outside the reviewed repository. Rendering a temporary copy is allowed; regeneration into the repository is not. Save a report or apply corrections only when the user requests that additional work.

## 1. Establish scope and authority

Resolve the target from the user's path, files, or repository reference. Ask only if the available context cannot establish it. Read applicable `AGENTS.md` / `CLAUDE.md`, the root `README.md`, and the documentation index (such as `docs/README.md`). Record the reviewed revision and any included working-tree changes so citations describe the same state.

Build a source map with each artifact's role, authority, approval/version status, and relationships:

- Primary requirements: original RFP, contractual scope, acceptance criteria, and approved amendments.
- Architecture and decisions: architecture documents, all relevant ADRs and their index, assumptions, and pending decisions/actions.
- Operational models: runbooks, process models, diagram sources, exported images, and their indexes.
- Deliverables: client-facing Word/PDF documents and the sources actually used to assemble them.
- Supporting evidence: coverage matrices, QA plans/results, build specifications, scope notes, API contracts, original source material, and implementation evidence when relevant.

Follow the project's stated source hierarchy. Where authority or approval is unclear, record that uncertainty instead of inventing precedence. Internal documents and coverage matrices are claims to check; agreement with them alone does not confirm a requirement or capability. Use Git history only to resolve a specific provenance question, retaining later authorized decisions.

Distinguish a deliverable's intended contents from complementary records. A nearby document is not automatically required inside the Word compilation. Expand beyond the named set only to validate relevant claims and dependencies.

Done when the source map identifies the requested artifacts, original requirements, compilation boundaries, and any missing or inaccessible material.

## 2. Check coverage, consistency, and decision status

Read every in-scope document in full, including tables, notes, appendices, and relevant linked records. Track each artifact as read, partially read, or unavailable; search results and summaries alone do not constitute complete analysis.

Trace each original in-scope requirement to its architectural treatment, decision, and operational flow. Include every priority unless an authorized scope boundary excludes it: priority does not establish a delivery phase. Compare against the original requirements, not just the coverage matrix. Distinguish an omission from a deliberate divergence already awaiting decision.

Check across architecture, ADRs, models, runbooks, and pending records:

- Responsibilities, systems of record, data ownership, trust boundaries, and handoffs between services.
- End-to-end flows: prerequisites, success, partial failure, duplicated/reordered events, retries, recovery, reconciliation, and manual operations where applicable.
- Ambiguous rules: spell out the competing interpretations and the consequence of each.
- Indispensable dependencies omitted or described as resolved without evidence.
- Repeated decisions with conflicting versions, stale references/links, identifiers, and counts.

Separate requirements, confirmed decisions, supplier proposals, client approvals, assumptions awaiting validation, contracts awaiting delivery, implementation/configuration tasks, planned tests, and demonstrated results. Check each assumption's owning ADR, status, statement, limits, consequence if wrong, and reversal cost/timing. Verify that pending records capture genuinely outstanding decisions and contracts without duplicating answered questions.

An explicitly qualified proposal is not a false assertion. A settled decision can still have configuration and testing work ahead. A known gap adequately tracked is not a new finding unless a separate defect is demonstrated, such as an incompatible guarantee elsewhere. Preserve authorized decisions when recommending corrections.

Done when every in-scope requirement and artifact is accounted for and candidate contradictions have been checked against their qualifications and current decisions.

## 3. Validate technical claims

Prioritize claims that support architecture decisions, guarantees, or contractual commitments. Read the actual relevant contract or implementation evidence before relying on supporting prose.

For current library, SDK, API, CLI, or cloud/service documentation, use Context7: resolve the library ID first (unless an exact ID was supplied), select the relevant version, then query one concept per call. Use concrete official vendor documentation when Context7 is unavailable or insufficient; disclose unresolved checks when neither source is accessible. Cite the specific source and relevant version/date rather than a generic documentation homepage.

Distinguish documented platform capability, availability in the contracted plan/license, project configuration, and behavior demonstrated by this implementation. Missing examples or remembered limitations do not prove impossibility. When evidence is insufficient, name the missing fact and the check that would settle it.

For API claims, identify the contract version, operation, method/path, and request/response datatype. Distinguish reading an existing object or code from creating, issuing, or changing it; use only fields and semantics supported by the actual contract.

For commerce architecture and integration reviews, read [references/commerce-integrations.md](references/commerce-integrations.md) and apply the relevant domain checks. The checklist guides investigation; it does not supply project requirements or vendor facts.

Done when each consequential technical claim has supporting evidence, contradicting evidence, or an explicit unresolved validation.

## 4. Check deliverable fidelity and presentation

For Word/PDF compilations or source/export diagrams, read [references/deliverable-fidelity.md](references/deliverable-fidelity.md). Compare with the established source map, extract structural content, render outside the repository, and inspect every in-scope rendered page and diagram.

For other formats, inspect their actual reading surface where available. Check missing or stale content, changes of meaning or approval status, tables, notes, references, titles, numbering, captions, navigation, and diagram legibility. Presentation defects count when they mislead or impede use; stylistic alternatives remain editorial suggestions.

Done when semantic fidelity and actual visual coverage are recorded separately, with exact uninspected elements identified. Failed rendering narrows the review claim; it does not prevent reporting evidenced semantic findings.

## 5. Validate and consolidate findings

Read [validate-finding](../validate-finding/SKILL.md) and apply its evidence and disconfirmation rubric to each candidate. For documentation, the documented rule, requirement, contract, or source comparison establishes expected meaning; a direct contradiction can be proven without a runtime reproduction.

Read the full context and related documents, search for qualifications and explicit decisions, and state the concrete impact. Group occurrences with the same root cause and cite every affected location. Zero findings is valid. Personal preferences alone are editorial suggestions.

Map the helper's dispositions to this report:

- `bug`: confirmed documentation error, including a proven contradiction or omission against an established obligation.
- `possible issue`: unresolved risk or ambiguity; identify what is known and what evidence would settle it.
- `recommendation`: editorial or optional improvement, clearly separate from defects.
- `false-positive`: discard, retaining its neutralizing evidence in working notes if useful.

A demonstrable ambiguity can be confirmed even when the intended rule is unknown: quote the wording, show the plausible interpretations, and explain their different effects. Label it a confirmed ambiguity without claiming either interpretation is the approved one.

Recommend the smallest correction that resolves the problem within each document's responsibility. Done when every candidate has an evidence-backed disposition and duplicate causes have been merged.

## 6. Deliver the report

Lead with the highest-impact confirmed findings. Separate confirmed errors/ambiguities, inconclusive validations, and editorial suggestions. Classification, severity, and confidence are independent:

- **Critical / crítica**: following the documentation could cause severe security/data harm or invalidate a fundamental contractual or go-live decision.
- **High / alta**: materially changes scope, responsibilities, financial behavior, or a core operational flow.
- **Medium / média**: a localized gap or ambiguity with concrete implementation or operational cost.
- **Low / baixa**: limited impact on comprehension, traceability, or presentation.

Give each finding a stable ID (`F-001`, `F-002`, …) and include:

- Concrete title, type, severity, and confidence.
- Exact file and line/section; for Word/PDF, verified rendered page and heading when available. Without verified pagination, cite the heading/table and state that the page is unverified.
- Short evidence excerpt plus the conflicting passage, original requirement, or concrete technical source.
- Explanation of the problem and its practical impact.
- Recommended correction and all affected documents.
- Any remaining validation and the fact needed to resolve it.

End with documents and sources actually analyzed, requirement coverage, rendered/inspected page and diagram coverage, and remaining limitations. Distinguish unavailable sources from reviewed sources. If relevant material remains unexamined, report only conclusions supported by the covered scope; do not certify the entire documentation as correct.
