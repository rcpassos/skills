---
name: validate-finding
description: Validate suspected issues, bugs, code review findings, analyzer warnings, user reports, audit findings, or recommendations by checking evidence, reproduction, impact, and intent. Use when you need to decide whether a claim is a bug, possible issue, false-positive, or recommendation before reporting it, changing code, or creating follow-up work. Also the verification rubric for audit-rcp and code-review-rcp.
---

# Validate Finding

Restate the claim in one sentence, establish the expected behaviour (code, tests, docs, contracts, framework rules), trace the actual path, and hunt for disconfirming evidence — guards, validation, config, callers, tests, framework semantics — before labelling. Split a multi-part claim and classify each part.

## Labels and evidence

Exactly one label per claim:

- `bug` — implementation contradicts expected behaviour with a credible failing path. Needs a reproduction, a test scenario that follows from the code, a source trace to invalid state or a missing check, or a violated documented rule. Static proof is enough; live repro is optional.
- `false-positive` — contradicted by a guard, constraint, flag, caller contract, passing test, framework semantics, or intentional product behaviour. Cite the neutralising evidence. Low impact alone never makes a claim a false positive.
- `recommendation` — a valid improvement with no broken behaviour (readability, perf without demonstrated harm, optional hardening, taste). Say what improves and why it isn't a defect.
- `possible issue` — plausible but unproven. Name the missing fact that would settle it.

## Output

```text
Classification: bug | possible issue | false-positive | recommendation
Confidence: high | medium | low
Claim: <one sentence>
Evidence: <code/test/doc/runtime facts, file:line>
Impact: <user, data, security, reliability, maintainability, or none proven>
Next action: <fix, test, investigate, ignore>
```
