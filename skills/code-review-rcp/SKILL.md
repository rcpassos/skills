---
name: code-review-rcp
description: Run the standard two-axis code-review (Standards + Spec) on a PR, branch, or diff, add a correctness/security/architecture pass, then verify the findings and sort the survivors into "Take it" (real bugs, blockers, important improvements) and "Leave it" (minor or YAGNI). Use when the user wants a triaged, actionable code review — e.g. "/code-review-rcp <PR-link>" or "review this branch and tell me what's worth fixing".
disable-model-invocation: true
---

A wrapper around the `code-review` skill: run it unchanged, add a second review pass for the things Standards and Spec don't cover, then verify and triage everything.

## Process

### 1. Run the base review

Invoke the **`code-review`** skill with whatever the user passed through (a PR link, branch, commit SHA, tag, `main`, `HEAD~5`, …) and let it complete its full process — pinning the fixed point, locating the spec, spawning the Standards and Spec sub-agents, and aggregating.

If the user gave a PR link, put the PR's code at `HEAD` first — `code-review` diffs `<fixed-point>...HEAD`, so running it from any other branch reviews the wrong code:

- Read the PR's refs: `gh pr view <ref> --json headRefName,headRefOid,baseRefName`.
- If `HEAD` isn't already `headRefOid`: with a clean working tree, run `gh pr checkout <ref>`; with uncommitted work, ask before switching, or review from a separate `git worktree`.
- Fetch the base (`git fetch origin <baseRefName>`) and hand `origin/<baseRefName>` to `code-review` as the fixed point.

Do not re-implement or shortcut any part of that skill.

Keep the base skill's raw output — it's an input to step 3, not the deliverable.

### 2. Run the review checklist pass

Reusing the same fixed point, diff and spec the base skill pinned, review the change against the checklist below. Run this as a `general-purpose` sub-agent (or in parallel sub-agents, one per group, on a large diff) so it doesn't inherit the base review's framing — pass it the diff command, the commit list, the spec, and the checklist verbatim.

Read past the diff: open the surrounding file and the callers of anything changed. Most of these only show up in the code the diff doesn't touch.

**Correctness**

- **Hidden bugs** — logic that is wrong in a way the tests and the happy path don't reveal: off-by-one, inverted condition, wrong variable, mutation of shared state, unawaited promise, swallowed error, resource never released, race between concurrent paths.
- **Edge cases** — empty, single-element, and very large inputs; `null`/`undefined`/`NaN`/`0`/`""` where truthiness is tested; boundary values; duplicates; unicode; timezones and DST; concurrent or repeated calls (is this idempotent?); partial failure and retry; network timeout; pagination limits.
- **Incorrect assumptions** — things the code takes for granted that the rest of the system doesn't guarantee: a field that is actually optional or nullable, ordering that isn't guaranteed, uniqueness that isn't enforced, a call that can fail or return partial data, a cache or clock assumed fresh, a type assertion or non-null assertion covering an unverified shape, an invariant enforced in one call site but not the others.

**Security**

- Untrusted input reaching a query, command, path, template, or deserializer without validation (injection, path traversal, SSRF).
- Missing or wrong authentication/authorization on a new endpoint, handler, or resource — especially object-level access checks (can user A pass user B's id?).
- Secrets, tokens, or PII in code, logs, error messages, or client-visible responses.
- Weak or hand-rolled crypto, randomness used for security purposes, unsafe redirects, missing rate limiting on expensive or auth-adjacent paths.
- Dependencies added or bumped in the diff: unfamiliar, unmaintained, or known-vulnerable.

**Architecture**

- **Architectural problems** — a change that fights the shape of the codebase: business logic leaking into a transport/UI layer, a dependency pointing the wrong way (inner layer importing an outer one), a new cross-module coupling or cycle, state duplicated in two places that can now drift, an abstraction that leaks its implementation, a boundary crossed that the codebase deliberately keeps closed, or a pattern invented here when the repo already has one for this.
- Note migration, backward-compatibility, and data-shape changes that break existing callers or stored data.

**Against the spec**

- **Missing requirements** — anything the spec, issue, or PR description asked for that isn't in the diff, or is only partially done. Quote the spec line. Include the implicit ones the spec implies: error handling, tests, docs, telemetry, and updates to the callers of anything whose signature changed.
- **Scope creep** — behaviour, config, refactors, or dependencies in the diff that nothing asked for. Say whether each is harmless, or a risk that should be split into its own change.

If there's no spec, say so and skip the last group.

### 3. Double-check every finding

Pool the findings from steps 1 and 2, then go back to the actual code and confirm each one against the diff and the surrounding file. Drop anything that doesn't survive:

- the code doesn't actually do what the finding claims (misread hunk, missing context outside the diff);
- the case is already handled elsewhere — a guard, a type, a validation layer, an existing test;
- the "violation" is enforced or auto-fixed by tooling (formatter, linter, compiler);
- it's a restatement of another finding — merge duplicates into one;
- it's a matter of taste with no defensible cost.

Classify each remaining finding with the `validate-finding` skill's rubric and evidence standards: `bug` and `recommendation` go to step 4; `false-positive` is dropped. A `possible issue` you can't settle either way is not a survivor — leave it out of both lists, and mention it in one line after the counts only when it materially affects confidence in the review.

### 4. Report two lists

Present only the surviving findings, in exactly two sections:

**Take it** — real bugs, correctness, security, and data-loss issues, blockers, architectural problems worth fixing now, missing requirements, and improvements important enough to be worth doing before merge. Order by severity, worst first. For each: one-line claim, `file:line`, the concrete failure it causes, and the fix.

**Leave it** — minor improvements, style nits, speculative generality, YAGNI, harmless scope creep, and anything whose cost outweighs its benefit right now. Keep each to a single line, and say briefly why it isn't worth doing.

End with one line: counts per list, plus how many findings were dropped as false positives and how many were left unverified.

Do not pad either list. An empty **Take it** is a fine outcome and should be stated plainly.
