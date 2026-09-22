---
name: issue-check-rcp
description: Validate a GitHub issue before anyone starts work on it — is the problem real, is it worth solving, is it clear enough to build, what could go wrong, and roughly how long would it take. Grounded in the actual codebase, not just the issue text. Use when the user asks to validate, sanity-check, size, or estimate the risk of an issue, e.g. "let's validate <github-issue>, is it useful and valid? what's the risk and size?".
disable-model-invocation: true
---

Answer four questions about one issue, in this order: **is it valid**, **is it useful**, **what's the risk**, **how big is it**. Every answer is grounded in the repo — read the code before judging.

This skill does not implement anything and does not open a PR. Its output is a verdict.

## Process

### 1. Fetch the issue

Take the issue reference the user gave (URL, `#123`, or `owner/repo#123`).

- Prefer `gh issue view <ref> --comments` (or the workflow in `docs/agents/issue-tracker.md` if the repo documents one).
- Read the **whole thread**, not just the opening post — the real requirement, the objections, and the "actually we decided X" often live in the comments.
- Note linked issues and PRs, labels, milestone, and who's asking.

If the issue can't be fetched, say so and ask for the text rather than guessing.

Restate the issue in two or three sentences of your own words: **what is being asked for, and why**. If you can't do that from what's written, that's the first finding — record it and keep going.

### 2. Ground it in the codebase

Find the code the issue is actually about before judging any of it. Look for:

- The area that would change, and the files most likely to be touched.
- Whether the problem **still exists** — for a bug, trace the code path or write a quick check; the fix may already be in, or the code may have moved.
- Whether it's **already solved elsewhere**, partly done, or duplicated by another issue or an open PR.
- The blast radius: callers of anything that would change, public API/schema/config surface, stored data shape, other teams' or clients' dependence on current behaviour.
- What the codebase's own conventions say about how this kind of thing gets built here.
- Tests that cover this area today — their absence is a risk, not a saving.

Use sub-agents for the search when the repo is large; cite `file:line` for anything you assert.

### 3. Validity

Is this a well-formed, real, actionable issue? Judge:

- **Real** — the problem reproduces, or the need is genuine; not already fixed, not a misunderstanding of how the feature works.
- **Clear** — a reader can tell what "done" means. Are there acceptance criteria? If not, propose them.
- **Right-shaped** — one issue, not five smuggled into one; a problem statement rather than a pre-committed solution; scoped to something a person can finish.
- **Unambiguous** — list every question that must be answered before work starts, and mark which are blocking.
- **Assumptions** — name what the issue takes for granted that the code doesn't support.

Verdict: **Valid** / **Valid with changes** (say exactly which) / **Invalid** (say why — duplicate, already fixed, works as designed, out of scope).

### 4. Usefulness

Is it worth doing at all?

- Who benefits, how many of them, and how often does this bite?
- What is the cost of _not_ doing it — workaround exists, or people are blocked?
- Does it move something the project already says it cares about, or is it a drive-by preference?
- Cheaper alternatives: docs, config, a smaller fix, or declining it.

Verdict: **Worth doing now** / **Worth doing later** / **Not worth doing** — with the one-line reason.

### 5. Risk

What could go wrong if this is built as described? Cover, and skip what doesn't apply:

- **Breaking change** — API, schema, config, CLI, stored data, or behaviour existing users rely on. Migration needed?
- **Blast radius** — how many call sites, modules, or services move; how reversible the change is.
- **Security & data** — new untrusted input, new authz surface, PII, secrets, anything touching auth or payments.
- **Correctness hazards** — concurrency, ordering, idempotency, partial failure, performance at real data volume.
- **Unknowns** — parts nobody can size yet, external dependencies, decisions the issue leaves open.
- **Test coverage** — is there a safety net here, or would this be built blind?

Give an overall **Low / Medium / High**, name the single biggest risk, and say what would lower it (a spike, a feature flag, splitting the issue, a decision from a named person).

### 6. Size

A **rough time range** for one competent engineer already familiar with the repo — e.g. "half a day to two days", "3–5 days". Always a range, never a single number.

State it with:

- The assumptions it rests on, and what would blow it up (the range is only as good as these).
- The work included: implementation, tests, migration, docs, review, rollout.
- Where the uncertainty is concentrated.
- If the range spans more than ~4×, say the issue is too unclear to size and name what must be decided first.

### 7. Report

Short and skimmable, in this order:

1. **One-line verdict** — e.g. "Valid and worth doing; medium risk; 2–4 days" — then the restatement from step 1.
2. **Validity** — verdict, then blocking questions as a list.
3. **Usefulness** — verdict and reason.
4. **Risk** — level, biggest risk, mitigations.
5. **Size** — range, assumptions, what would blow it up.
6. **Before starting** — the concrete next actions: decisions to get, acceptance criteria to add, how to split it if it should be split.

Be direct. "This issue is already fixed in `x.ts:112`, close it" is a better answer than a balanced essay. If the honest answer is "can't tell yet", say that and name exactly what is missing.
