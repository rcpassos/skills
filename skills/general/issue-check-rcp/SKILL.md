---
name: issue-check-rcp
description: Validate a GitHub issue before anyone starts work on it — is the problem real, is it worth solving, is it clear enough to build, what could go wrong, and roughly how long would it take. Grounded in the actual codebase, not just the issue text. Use when the user asks to validate, sanity-check, size, or estimate the risk of an issue, e.g. "let's validate <github-issue>, is it useful and valid? what's the risk and size?".
disable-model-invocation: true
---

Answer four questions about one issue, in order: **valid**, **useful**, **risk**, **size**. Every answer is grounded in the repo — read the code before judging. The output is a verdict; implement nothing and open no PR.

## Process

### 1. Fetch the issue

`gh issue view <ref> --comments` (or the workflow in `docs/agents/issue-tracker.md` if present). Read the whole thread — the real requirement and the "actually we decided X" often live in the comments. Note linked issues and PRs, labels, milestone. If it can't be fetched, ask for the text.

Restate what is asked and why in two or three sentences. If you can't, that is the first finding.

### 2. Ground it in the codebase

Cite `file:line` for every assertion; use sub-agents on a large repo. Establish:

- The code that would change.
- Whether the problem still exists — trace the path or run a quick check; it may already be fixed, or the code moved.
- Whether it's already solved, partly done, or duplicated by another issue or open PR.
- Blast radius: callers, public API/schema/config, stored data, external consumers.
- Existing tests for the area — their absence is a risk.

### 3. Validity

Judge whether it is real, has a clear "done" (propose acceptance criteria if missing), is one right-sized problem rather than a pre-committed solution, and what it assumes that the code doesn't support. List every open question, marking which block work.

Verdict: **Valid** / **Valid with changes** (say which) / **Invalid** (duplicate, already fixed, works as designed, out of scope).

### 4. Usefulness

Who it helps and how often, the cost of not doing it, whether it serves a stated project goal, and cheaper alternatives (docs, config, a smaller fix, declining).

Verdict: **Worth doing now** / **Worth doing later** / **Not worth doing**, with a one-line reason.

### 5. Risk

Weigh breaking changes and migrations, blast radius and reversibility, security and data exposure, correctness hazards, unknowns, and test coverage.

Verdict: **Low** / **Medium** / **High**, the single biggest risk, and what would lower it (a spike, a flag, a split, a decision from a named person).

### 6. Size

A time range for one competent engineer familiar with the repo — always a range, never a single number. Include implementation, tests, migration, docs, review, and rollout. State the assumptions, what would blow it up, and where the uncertainty sits. A range wider than ~4× means the issue is too unclear to size — name what must be decided first.

### 7. Report

1. **One-line verdict** — e.g. "Valid and worth doing; medium risk; 2–4 days" — then the restatement.
2. **Validity** — verdict, then blocking questions.
3. **Usefulness** — verdict and reason.
4. **Risk** — level, biggest risk, mitigations.
5. **Size** — range, assumptions, what would blow it up.
6. **Before starting** — decisions to get, acceptance criteria to add, how to split it.

Be direct: "Already fixed in `x.ts:112`, close it" beats a balanced essay. If the honest answer is "can't tell yet", say so and name what's missing.
