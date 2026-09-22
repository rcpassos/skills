---
name: validate-finding
description: Validate suspected issues, bugs, code review findings, analyzer warnings, user reports, audit findings, or recommendations by checking evidence, reproduction, impact, and intent. Use when you need to decide whether a claim is a bug, possible issue, false-positive, or recommendation before reporting it, changing code, or creating follow-up work. Also the verification rubric for audit-rcp and code-review-rcp.
---

# Validate Finding

## Overview

Use this skill to classify a suspected finding with enough evidence that another engineer can trust the label. Prefer source-backed reasoning over intuition, and separate broken behavior from optional improvement.

## Workflow

1. Restate the claim in one sentence.
2. Identify the expected behavior from code, tests, docs, user request, product rules, API contracts, security expectations, or framework conventions.
3. Inspect the actual behavior path. Follow inputs, branches, state changes, side effects, persistence, external calls, and rendered output as far as needed.
4. Look for disconfirming evidence. Check guards, validation, configuration, migrations, policies, tests, callers, feature flags, and framework behavior that may make the claim impossible.
5. Classify the finding using the rubric below.
6. State the confidence, evidence, impact, and the smallest next action.

## Classification Rubric

Use exactly one primary category:

- `bug`: The implementation contradicts expected behavior and there is a credible failing path. Use for crashes, data loss, security bypasses, wrong output, broken permissions, incorrect persistence, user-visible regressions, or contract violations.
- `possible issue`: The concern is plausible but not proven. Use when evidence is incomplete, behavior depends on missing runtime state, impact is unclear, or reproduction requires confirmation.
- `false-positive`: The claim is contradicted by the codebase, tests, configuration, runtime constraints, framework semantics, or product requirements. Explain the evidence that neutralizes the concern.
- `recommendation`: The claim is valid as an improvement but does not show broken behavior. Use for readability, maintainability, performance tuning without demonstrated harm, architecture preference, UX polish, or optional hardening.

If a claim contains multiple concerns, split them and classify each one separately.

## Evidence Standards

For `bug`, require at least one of:

- A concrete reproduction path.
- A failing or missing test scenario that follows directly from code.
- A source-level trace showing invalid state, incorrect branching, missing authorization, unsafe persistence, wrong type/shape, or incorrect external contract usage.
- A documented requirement or framework rule that the implementation violates.

For `false-positive`, require at least one of:

- A guard, validation rule, policy, type constraint, database constraint, feature flag, or caller contract that prevents the alleged failure.
- A passing test that covers the alleged behavior.
- Framework or library semantics showing the concern cannot occur.
- Product requirements showing the behavior is intentional.

For `recommendation`, make clear what improves and why it is not currently a defect.

For `possible issue`, name the missing evidence that would upgrade or dismiss the finding.

## Output Format

When reporting the result, keep it compact:

```text
Classification: bug | possible issue | false-positive | recommendation
Confidence: high | medium | low
Claim: <one-sentence restatement>
Evidence: <specific code/test/doc/runtime facts>
Impact: <user, data, security, reliability, maintainability, or none proven>
Next action: <fix, test, investigate, ignore, or convert to recommendation>
```

For code review findings, lead with the category and severity. Include file and line references when available.

## Guardrails

- Do not classify style preference, naming preference, or speculative architecture taste as a bug.
- Do not call something a false-positive just because it is low impact; low-impact defects can still be bugs.
- Do not require live reproduction when static evidence is enough to prove a failing path.
- Do not recommend code changes until the category is clear.
- Do not bury uncertainty. If the decisive fact is missing, classify as `possible issue` and state what to verify.
