---
name: code-review-rcp
description: Review a PR, branch, or diff for standards, spec, and risk; verify findings and carry decisions across follow-up reviews of fixes.
disable-model-invocation: true
---

Review a pinned revision along three axes (Standards, Spec, Risk) and report only verified findings. Follow-up reviews use the previous review record to check fixes. Leave implementation unchanged unless the user asks for fixes; the only file you write is the review record.

## The bar

A finding has a concrete cost: a bug, a security or data-loss issue, a missing or wrong requirement, or a maintainability cost — code that makes future change harder or riskier (duplicated logic that will drift, coupling across layers, a misleading name or API, a risky path without tests). Style, nits, YAGNI, taste, and harmless scope creep fall below it; below-bar findings go unreported and uncounted.

## Process

### 1. Pin the revision and choose the mode

Resolve target and base from what the user gave (PR link, branch, SHA, tag, prior record); ask only when neither is clear. A PR is reviewed at its head commit against `origin/<baseRefName>` (`gh pr view <ref> --json headRefName,headRefOid,baseRefName,body`); when uncommitted work blocks a checkout, ask or use a `git worktree`.

Pin base, head, and merge-base to SHAs and use only those in every diff, log, and brief; read surrounding code at that revision. Uncommitted changes are covered only when the user includes them, as a recorded snapshot.

Look for a matching review record, supplied by the user or under `.scratch/code-review/`, matched by repository and PR or branch identity. **Initial review** covers the full diff. **Follow-up** applies when the user asks to check fixes or continue a recorded review: read [references/follow-up.md](references/follow-up.md). With no usable record, say so and do an initial review.

Done when the SHAs and mode are recorded.

### 2. Gather spec and standards

**Spec**: the user's source, the PR body and linked issues (`gh issue view <ref> --comments`, or `docs/agents/issue-tracker.md` if present), issue refs in commits, and matching specs under `docs/`, `specs/`, or `.scratch/`. Record acceptance criteria, later decisions, and exclusions; raise conflicting requirements as questions. None found → ask; if none exists, skip the Spec axis.

**Standards**: `CODING_STANDARDS.md`, `CONTRIBUTING.md`, `AGENTS.md` / `CLAUDE.md`, `docs/adr/`, and lint/format config (what tooling already enforces).

Blocking threshold: the user's, else P0/P1.

### 3. Review

Size the diff with `git diff --stat`:

- **Inline**: up to ~300 changed lines in one subsystem. Review it yourself, all three axes in one pass, classifying each candidate with [validate-finding](../validate-finding/SKILL.md) as you go.
- **Sliced**: anything larger. Split by subsystem so each changed file belongs to one slice, and dispatch one sub-agent per slice in parallel, each applying all three axes. Where the harness lets you choose, slice reviewers can run on a faster model; step 4 re-checks what blocks.

Each slice brief carries the SHAs and the slice's diff command, the contracts, exclusions, and prior decisions, the bar and the axes below verbatim, and ends: "Read surrounding code and affected callers. Classify each candidate with the validate-finding skill and return only `bug` and `recommendation` findings that clear the bar, each with file:line, trigger, expected versus actual, the contract it breaks, evidence (reproduction or complete source trace), how this change introduces or exposes it, and confidence. Also return coverage gaps, leads that cross into other slices, and any `possible issue` with the fact that would settle it."

**Standards**: breaches of documented rules (cite file and rule) and code smells labelled "possible <smell>". A documented standard that endorses the pattern suppresses the smell; skip what tooling enforces.

**Spec**: quote the spec line per finding.

- Missing requirements grounded in the spec, a documented contract, or established behaviour. A missing test, doc, or log line is a defect only when such evidence requires it.
- Wrong implementation: looks done, behaves differently from the spec.
- Scope creep that adds risk or maintenance burden; say whether to split it out.

**Risk**

- **Correctness**: edge cases, assumptions the system doesn't guarantee, invariants enforced at only some call sites.
- **Security**: untrusted input reaching a sink, missing object-level authz on new surface, secrets or PII (location and type, never the value), deps added or bumped.
- **Architecture**: layer leaks, new coupling, an invented pattern where one exists, compat breaks for callers or stored data.

Done when every slice is reviewed on every applicable axis, returning findings or an explicit none, plus coverage gaps.

### 4. Verify

Merge findings by root cause and chase cross-slice leads. Reviewers have already classified their findings; re-check yourself only P0/P1 candidates, low-confidence findings, and claims spanning slices, against the pinned code, the base revision when causation is unclear, and disconfirming guards, callers, and tests. A configured tool alone doesn't prove a correctness or security issue is prevented.

Pre-existing defects count when the change makes them reachable or materially worse; report any other serious incidental discovery separately, outside the gate. An unsettled `possible issue` stays a verification gap naming the missing fact. Done when every finding has evidence and a disposition.

### 5. Report

Rank confirmed defects by impact and reachability: **P0** critical, **P1** fix before merge, **P2** moderate, **P3** low. Severity is separate from classification. List maintainability recommendations separately as optional improvements. Each finding: stable ID, severity, claim, `file:line`, trigger and impact, evidence, suggested fix. Follow-ups report status changes and new findings only.

Gate: **met** (no open blocking finding, required checks pass, no material gap), **not met** (open findings at the threshold), or **inconclusive** (missing evidence could change the decision). Deferral alone doesn't resolve a blocker; user-approved exceptions stay explicit.

End with counts by severity, optional improvements, false positives dropped, coverage limits, and "no spec" when applicable. Stop after this scoped pass.

### 6. Save the review record

Update the matching record or create `.scratch/code-review/<target>.md` (filesystem-safe PR or branch id), link it in the report, and keep it out of commits. If the user forbids writes, give it inline. Keep it compact and current:

- Target identity, base/head/merge-base SHAs or snapshot, mode and covered scope.
- Spec and standards sources, acceptance criteria, exclusions, blocking threshold, approved exceptions.
- Findings with stable IDs (`R1`, `R2`, …; new IDs only for new root causes): root cause, classification, severity, location, evidence, status (open, resolved, rejected, deferred), and who decided and why for any acceptance or deferral.
- Checks run and results, verification gaps, gate outcome, and the revision each resolution was verified against.

Keep enough rejected and resolved evidence that settled claims aren't reopened without cause; omit nits.
