---
name: code-review-rcp
description: Review a PR, branch, or diff along three axes — Standards (repo conventions and code smells), Spec (does it do what the issue asked), and Risk (correctness, security, architecture) — then verify the findings and sort the survivors into "Take it" (real bugs, blockers, important improvements) and "Leave it" (minor or YAGNI). Use when the user wants a triaged, actionable code review — e.g. "/code-review-rcp <PR-link>" or "review this branch and tell me what's worth fixing".
disable-model-invocation: true
---

Review the diff between `HEAD` and a fixed point along three axes, each in its own parallel sub-agent so none inherits another's framing, then verify and triage into two lists.

## Process

### 1. Pin the fixed point

The user passes a PR link, branch, SHA, tag, `main`, `HEAD~5`, … — if nothing, ask.

For a PR link, put the PR's code at `HEAD`:

- `gh pr view <ref> --json headRefName,headRefOid,baseRefName,body`.
- If `HEAD` isn't `headRefOid`: clean tree → `gh pr checkout <ref>`; uncommitted work → ask, or review from a `git worktree`.
- `git fetch origin <baseRefName>`; the fixed point is `origin/<baseRefName>`.

Capture for every sub-agent: `git diff <fixed-point>...HEAD` (three-dot, against the merge-base) and `git log <fixed-point>..HEAD --oneline`.

Done when the fixed point resolves and the diff is non-empty; otherwise stop before any sub-agent runs.

### 2. Gather spec and standards

**Spec** — the first found: the PR body and linked issue; issue refs in commit messages (`gh issue view <ref> --comments`, or `docs/agents/issue-tracker.md` if present); a path the user passed; a spec under `docs/`, `specs/`, or `.scratch/` matching the branch. None found → ask; if there is none, skip the Spec axis.

**Standards** — `CODING_STANDARDS.md`, `CONTRIBUTING.md`, `AGENTS.md` / `CLAUDE.md`, `docs/adr/`, and lint/format config (to know what tooling already enforces).

### 3. Run the three axes in parallel

One message, three `general-purpose` sub-agents. Each gets the diff command, the commit list, and its axis below verbatim. On a large diff, split Risk into one sub-agent per group.

Every brief ends: "Read past the diff: open the surrounding file and the callers of anything changed. Report each finding with `file:line`, the quoted hunk, and the concrete failure or cost. Under 400 words."

**Standards** — pass the standards files. Report every breach of a documented rule (cite file and rule), plus code smells. Label smells "possible <smell>" — only documented rules are hard violations. A documented standard that endorses a pattern suppresses the smell; skip what tooling enforces.

**Spec** — pass the spec; quote the spec line per finding.

- Missing requirements, including implicit ones: error handling, tests, docs, telemetry, callers of changed signatures.
- Wrong implementation — looks done, behaves differently from the spec.
- Scope creep — unrequested behaviour, config, refactors, or deps; say whether each is harmless or worth splitting out.

**Risk**

- **Correctness** — hidden bugs the happy path and tests miss; edge cases (empty/huge input, falsy values, boundaries, unicode, timezones, repeat/concurrent calls, partial failure); assumptions the system doesn't guarantee (nullability, ordering, uniqueness, freshness, unchecked casts, invariants enforced at only some call sites).
- **Security** — untrusted input reaching a query, command, path, template, or deserializer; missing authn/authz on new surface, especially object-level; secrets or PII in code, logs, or responses (report location and type, never the value); weak crypto, open redirects, missing rate limits; deps added or bumped in the diff.
- **Architecture** — changes that fight the codebase's shape (layer leaks, new coupling or cycles, drifting duplicated state, an invented pattern where one exists); compat breaks for callers or stored data.

### 4. Verify every finding

Pool the axes, merge duplicates, and confirm each against the diff and surrounding file. Drop misreads, cases already handled elsewhere (guard, type, validation, test), anything tooling enforces, and taste with no defensible cost.

Classify the rest with the `validate-finding` skill: `bug` and `recommendation` survive. An unsettled `possible issue` is not a survivor — mention it after the counts only if it materially affects confidence.

### 5. Report

**Take it** — real bugs, security and data-loss issues, blockers, missing requirements, and improvements worth doing before merge, worst first. Each: one-line claim, `file:line`, the failure it causes, fix.

**Leave it** — minor improvements, nits, YAGNI, harmless scope creep. One line each with why it can wait.

End with one line: counts per list, false positives dropped, findings left unverified, and "no spec" if the Spec axis was skipped. An empty **Take it** is a fine outcome — state it plainly.
