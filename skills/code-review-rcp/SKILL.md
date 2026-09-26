---
name: code-review-rcp
description: Review a PR, branch, or diff for standards, spec, and risk; verify findings and carry decisions across follow-up reviews of fixes.
disable-model-invocation: true
---

Review a pinned revision along three independent axes, then verify findings. Follow-up reviews use the previous review record to check fixes and affected behavior. Leave implementation unchanged unless the user asks for fixes; save only the review record by default.

## The bar

A finding has a concrete cost: a bug, a security or data-loss issue, a missing or wrong requirement, or a maintainability cost — code that makes future change harder or riskier (duplicated logic that will drift, coupling across layers, a misleading name or API, a risky path without tests). Style, nits, YAGNI, taste, and harmless scope creep fall below it; below-bar findings go unreported and uncounted.

## Process

### 1. Pin the revision and choose the mode

Resolve the comparison from the user's PR link, branch, SHA, tag, `main`, `HEAD~5`, or matching prior review record. Ask only when neither establishes the target and base.

For a PR link, put the PR's code at `HEAD`:

- `gh pr view <ref> --json headRefName,headRefOid,baseRefName,body`.
- If `HEAD` isn't `headRefOid`: clean tree → `gh pr checkout <ref>`; uncommitted work → ask, or review from a `git worktree`.
- `git fetch origin <baseRefName>`; the fixed point is `origin/<baseRefName>`.

Resolve the fixed point and head to commit SHAs, then record their merge-base SHA. Use those immutable values in every diff, log, and sub-agent brief: `git diff <merge-base-sha> <head-sha>` and `git log <base-sha>..<head-sha> --oneline`. Read surrounding files from that same revision, using an isolated checkout when the working tree differs. If the user includes uncommitted changes, capture a stable snapshot and identify it in the record; a HEAD-only review does not cover them.

Find a matching review record supplied by the user or under `.scratch/code-review/`. Match repository and PR or branch identity, not just a filename. **Initial review** covers the full diff. **Follow-up** applies when the user asks to check fixes or continue a recorded review; read [references/follow-up.md](references/follow-up.md). An explicit fresh review covers the full diff while retaining earlier decisions as context. With no usable record, state that limitation and perform an initial review.

Done when the snapshot, comparison, and mode are recorded. An empty initial diff ends with no changed code to review; an unchanged follow-up can still verify outstanding findings.

### 2. Gather spec and standards

**Spec** — start with the user-supplied source, then the PR body and linked issue, issue refs in commit messages (`gh issue view <ref> --comments`, or `docs/agents/issue-tracker.md` if present), and matching specs under `docs/`, `specs/`, or `.scratch/`. Include later decisions and explicit exclusions. Record the acceptance criteria and sources; surface conflicting requirements as questions rather than silently choosing one. None found → ask; if there is none, skip the Spec axis.

**Standards** — `CODING_STANDARDS.md`, `CONTRIBUTING.md`, `AGENTS.md` / `CLAUDE.md`, `docs/adr/`, and lint/format config (to know what tooling already enforces).

Use the user's blocking threshold; otherwise P0/P1 findings block. Done when the reviewers share the same contracts, exclusions, threshold, and known verification limits.

### 3. Run the three axes in parallel

For an initial review, dispatch one sub-agent per applicable axis, in parallel as capacity allows. Each gets the pinned checkout, diff command, commit list, contracts, prior decisions, and its axis below. On a large diff, split Risk by affected subsystem. For follow-up scope, use the reference above.

Every brief includes the bar verbatim and ends: "Read surrounding code and affected callers. For each candidate, give file:line, trigger, expected versus actual behavior, supporting contract, and a reproduction or complete source trace. Explain how this change introduces or exposes it. For recommendations, show the concrete maintenance cost. Keep each finding concise; retain material evidence and list coverage or verification gaps. Zero findings is valid."

**Standards** — pass the standards files. Report breaches of documented rules (cite file and rule) and code smells, each only where it clears the bar. Label smells "possible <smell>" — only documented rules are hard violations. A documented standard that endorses a pattern suppresses the smell; skip tooling-covered nits.

**Spec** — pass the spec; quote the spec line per finding.

- Missing requirements grounded in the spec, a documented contract, or established behavior. Error handling, tests, docs, telemetry, and caller updates count only when required by such evidence or a demonstrated failure; missing artifacts alone do not establish a defect.
- Wrong implementation — looks done, behaves differently from the spec.
- Scope creep — unrequested behaviour, config, refactors, or deps that add risk or maintenance burden; say whether to split each out.

**Risk**

- **Correctness** — hidden bugs the happy path and tests miss; edge cases (empty/huge input, falsy values, boundaries, unicode, timezones, repeat/concurrent calls, partial failure); assumptions the system doesn't guarantee (nullability, ordering, uniqueness, freshness, unchecked casts, invariants enforced at only some call sites).
- **Security** — untrusted input reaching a query, command, path, template, or deserializer; missing authn/authz on new surface, especially object-level; secrets or PII in code, logs, or responses (report location and type, never the value); weak crypto, open redirects, missing rate limits; deps added or bumped in the diff.
- **Architecture** — changes that fight the codebase's shape (layer leaks, new coupling or cycles, drifting duplicated state, an invented pattern where one exists); compat breaks for callers or stored data.

Done when each applicable axis returns findings or explicitly reports no findings, plus any material coverage gaps.

### 4. Verify every finding

Pool the axes and merge by root cause. Confirm each against the pinned code, surrounding paths, and contracts. Check disconfirming evidence: guards, validation, callers, and tests that actually exercise the claimed failure. Where safe and useful, run a targeted check and record its command and result; a complete static trace can suffice. An unrelated passing suite does not disprove a finding.

Classify with [validate-finding](../validate-finding/SKILL.md). Only `bug` and `recommendation` findings that clear the bar survive. Compare with the base revision when causation is uncertain. Existing defects count here when the change makes them reachable or materially worse; otherwise separate a serious incidental discovery from this review's findings and completion gate.

An unsettled `possible issue` stays a verification gap with the missing fact named. Drop false positives, nits already covered by tooling, and below-bar recommendations. A configured tool alone is not proof that a correctness or security issue is prevented. Done when each candidate has evidence and a disposition.

### 5. Report severity and completion

List confirmed defects by impact and reachability: **P0** critical, immediate action; **P1** high impact, fix before merge; **P2** moderate impact; **P3** low impact. Classification and severity are separate: being a bug does not automatically make it a blocker. List concrete maintainability recommendations separately as optional improvements.

Each finding includes a stable ID, severity, claim, `file:line`, trigger and impact, evidence, and a suggested fix. Follow-ups report status changes and new findings without repeating unchanged explanations.

State the gate outcome: **met**, **not met** (open findings at the blocking threshold), or **inconclusive** (material missing evidence or required checks could change the decision). The gate is met when blocking findings are resolved, relevant required verification passes, and no material verification gap remains. Disclose lesser uncertainties and deferred or lower-severity findings. Deferral alone does not resolve a blocker; any user-approved exception remains explicit. A finished review can have an unmet gate.

End with counts by severity, optional improvements, false positives dropped, coverage limits, and "no spec" when applicable. Stop after the scoped review and verification; another broad search needs a scope change, new evidence, or a user request. A met gate describes this revision and scope, not proof of bug-free code.

### 6. Save the review record

Update the matching record, or create `.scratch/code-review/<target>.md` using a filesystem-safe PR or branch identifier. Link it in the final report so another session can load it. If the user requires no writes, provide the record inline instead. Keep it out of commits unless requested.

Keep a compact current record containing:

- Repository and target identity, base/head/merge-base SHAs or snapshot identity, review mode and covered scope.
- Spec and standards sources, acceptance criteria, exclusions, blocking threshold, and any approved exceptions.
- Stable finding IDs (`R1`, `R2`, ...), root cause, classification, severity, location, evidence, and status: open, resolved, rejected, or deferred. Record the reason and who made any acceptance or deferral decision; assign new IDs only to new root causes.
- Checks and results, verification gaps, gate outcome, and the revision against which each resolution was verified.

Preserve enough rejected and resolved evidence to avoid reopening settled claims without cause; omit discarded nits. Done when the record matches the reported revision and decisions.
