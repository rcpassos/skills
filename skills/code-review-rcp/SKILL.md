---
name: code-review-rcp
description: Review a PR, branch, or diff along three axes — Standards (repo conventions and code smells), Spec (does it do what the issue asked), and Risk (correctness, security, architecture) — then verify the findings and sort the survivors into "Take it" (real bugs, blockers, important improvements) and "Leave it" (minor or YAGNI). Use when the user wants a triaged, actionable code review — e.g. "/code-review-rcp <PR-link>" or "review this branch and tell me what's worth fixing".
disable-model-invocation: true
---

Review the diff between `HEAD` and a fixed point along three independent axes, run in parallel sub-agents so none inherits another's framing, then verify and triage everything into two lists.

## Process

### 1. Pin the fixed point

The user passes a PR link, branch, commit SHA, tag, `main`, `HEAD~5`, … If they gave nothing, ask.

If they gave a PR link, put the PR's code at `HEAD` first — the review diffs `<fixed-point>...HEAD` and reads surrounding files, so it must run on the PR's branch:

- Read the PR's refs: `gh pr view <ref> --json headRefName,headRefOid,baseRefName,body`.
- If `HEAD` isn't already `headRefOid`: with a clean working tree, run `gh pr checkout <ref>`; with uncommitted work, ask before switching, or review from a separate `git worktree`.
- Fetch the base (`git fetch origin <baseRefName>`); the fixed point is `origin/<baseRefName>`.

Capture once, for every sub-agent:

- Diff command: `git diff <fixed-point>...HEAD` (three-dot, against the merge-base).
- Commit list: `git log <fixed-point>..HEAD --oneline`.

Done when `git rev-parse <fixed-point>` resolves and the diff is non-empty. A bad ref or empty diff stops here, before any sub-agent runs.

### 2. Gather the spec and standards

**Spec** — the first source found, in this order:

1. The PR body and any issue it links.
2. Issue references in commit messages (`#123`, `Closes #45`, GitLab `!67`) — fetch with `gh issue view <ref> --comments`, or the workflow in `docs/agents/issue-tracker.md` if the repo documents one.
3. A path the user passed.
4. A spec file under `docs/`, `specs/`, or `.scratch/` matching the branch name or feature.

If none turns up, ask the user. If there isn't one, the Spec axis is skipped and the report says so.

**Standards** — every repo file that documents how code should be written: `CODING_STANDARDS.md`, `CONTRIBUTING.md`, `AGENTS.md` / `CLAUDE.md`, `docs/adr/`, and lint/format config (to know what tooling already enforces).

### 3. Run the three axes in parallel

Send one message with three `general-purpose` sub-agent calls. Each gets the diff command, the commit list, and its section below pasted verbatim — sub-agents have no other access to it. On a large diff, split the Risk axis into one sub-agent per group.

Every brief ends with: "Read past the diff: open the surrounding file and the callers of anything changed. Report each finding with `file:line`, the quoted hunk, and the concrete failure or cost. Under 400 words."

#### Standards

Pass the standards files found in step 2. Report every place the diff breaks a documented rule (cite file and rule), plus any smell from the baseline below.

- **The repo overrides.** Where a documented standard endorses something the baseline flags, suppress the smell.
- **Smells are judgement calls.** Label each "possible <smell>"; only documented-rule breaches can be hard violations. Skip anything tooling enforces.

Smell baseline — each reads _what it is_ → _fix_:

- **Mysterious name** — name hides what it does or holds → rename; if no honest name comes, the design is murky.
- **Duplicated code** — same logic shape in more than one hunk or file → extract and call it from both.
- **Feature envy** — method reaches into another object's data more than its own → move it onto that data.
- **Data clump** — same few fields or params keep travelling together → bundle into one type.
- **Primitive obsession** — primitive or string standing in for a domain concept → give it a small type.
- **Repeated switch** — same `switch`/`if`-cascade on the same type at several sites → polymorphism, or one shared map.
- **Shotgun surgery** — one logical change scatters edits across many files → gather what changes together.
- **Divergent change** — one module edited for several unrelated reasons → split by reason to change.
- **Speculative generality** — abstraction, params, or hooks for needs the spec doesn't have → delete; inline until a real need shows.
- **Message chain** — long `a.b().c().d()` navigation → hide the walk behind one method.
- **Middle man** — class or function that only delegates → call the real target directly.
- **Refused bequest** — subclass ignores or overrides most of what it inherits → composition over inheritance.

#### Spec

Pass the spec contents. Skip this sub-agent when there is no spec. Quote the spec line for each finding.

- **Missing requirements** — anything asked for that the diff lacks or only partly does, including the implicit ones: error handling, tests, docs, telemetry, and updates to callers of any changed signature.
- **Wrong implementation** — requirements that look implemented but behave differently from what the spec says.
- **Scope creep** — behaviour, config, refactors, or dependencies nothing asked for. Say whether each is harmless or a risk worth splitting into its own change.

#### Risk

**Correctness**

- **Hidden bugs** — logic wrong in a way the tests and happy path don't reveal: off-by-one, inverted condition, wrong variable, mutation of shared state, unawaited promise, swallowed error, resource never released, race between concurrent paths.
- **Edge cases** — empty, single-element, very large inputs; `null`/`undefined`/`NaN`/`0`/`""` where truthiness is tested; boundary values; duplicates; unicode; timezones and DST; concurrent or repeated calls (idempotent?); partial failure and retry; network timeout; pagination limits.
- **Incorrect assumptions** — what the code takes for granted the system doesn't guarantee: optional/nullable field treated as present, ordering not guaranteed, uniqueness not enforced, call that can fail or return partial data, cache/clock assumed fresh, type or non-null assertion over an unverified shape, invariant enforced at one call site but not others.

**Security**

- Untrusted input reaching a query, command, path, template, or deserializer without validation (injection, path traversal, SSRF).
- Missing/wrong authn/authz on a new endpoint, handler, or resource — especially object-level checks (can user A pass user B's id?).
- Secrets, tokens, PII in code, logs, error messages, client-visible responses. Report only location and type, never the value.
- Weak or hand-rolled crypto, security-purpose randomness, unsafe redirects, missing rate limiting on expensive/auth-adjacent paths.
- Dependencies added or bumped in the diff: unfamiliar, unmaintained, or known-vulnerable.

**Architecture**

- A change that fights the codebase's shape: business logic leaking into a transport/UI layer, inner layer importing an outer one, new cross-module coupling or cycle, state duplicated in two places that can drift, leaky abstraction, a boundary the codebase keeps closed being crossed, a pattern invented when the repo already has one.
- Migration, backward-compat, and data-shape changes that break existing callers or stored data.

### 4. Double-check every finding

Pool the findings from all three axes, then go back to the actual code and confirm each one against the diff and the surrounding file. Drop anything that doesn't survive:

- the code doesn't actually do what the finding claims (misread hunk, missing context outside the diff);
- the case is already handled elsewhere — a guard, a type, a validation layer, an existing test;
- the "violation" is enforced or auto-fixed by tooling (formatter, linter, compiler);
- it's a restatement of another finding — merge duplicates into one;
- it's a matter of taste with no defensible cost.

Classify each remaining finding with the `validate-finding` skill's rubric and evidence standards: `bug` and `recommendation` go to step 5; `false-positive` is dropped. A `possible issue` you can't settle either way is not a survivor — leave it out of both lists, and mention it in one line after the counts only when it materially affects confidence in the review.

### 5. Report two lists

Present only the surviving findings, in exactly two sections:

**Take it** — real bugs, correctness, security, and data-loss issues, blockers, architectural problems worth fixing now, missing requirements, and improvements important enough to be worth doing before merge. Order by severity, worst first. For each: one-line claim, `file:line`, the concrete failure it causes, and the fix.

**Leave it** — minor improvements, style nits, speculative generality, YAGNI, harmless scope creep, and anything whose cost outweighs its benefit right now. Keep each to a single line, and say briefly why it isn't worth doing.

End with one line: counts per list, plus how many findings were dropped as false positives and how many were left unverified. If there was no spec, say so here.

Do not pad either list. An empty **Take it** is a fine outcome and should be stated plainly.
