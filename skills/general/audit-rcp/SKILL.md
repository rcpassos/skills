---
name: audit-rcp
description: Audit a whole codebase or GitHub repo for bugs, security holes, architecture, performance and reliability risks, and maintenance debt. For a PR or diff, use code-review-rcp.
disable-model-invocation: true
---

Read-only full-project audit: map the repo, review it in slices, and report only verified findings that clear the bar. Stay read-only unless the user asks for fixes; their instructions override this workflow.

## The bar

A finding has a concrete cost: a bug, a security or data-loss issue, a reliability or performance failure at realistic load, or a maintainability cost — code that makes future change harder or riskier (duplicated logic that will drift, coupling across layers, deprecated or EOL dependencies, a risky path without tests, a simplification with a concrete payoff). Style, nits, YAGNI, and taste fall below it; below-bar findings go unreported and uncounted.

## Process

### 1. Resolve the target

- **Local path**: confirm it resolves; that directory is the audit root.
- **GitHub reference** (URL, `owner/repo`): `git clone --depth 31 <url>` into a `mktemp -d` directory (31 commits feed the hotspot window) and record `git rev-parse HEAD`. Never push or touch the remote.
- Ambiguous between local and remote: ask.

### 2. Build the map

Skim once; the map drives slicing and goes into every brief:

- Purpose and conventions: `README.md`, `CONTEXT.md`, `AGENTS.md` / `CLAUDE.md`, `docs/adr/`.
- Runtime and locked dependency versions.
- Layout: entry points, layer boundaries, where auth, input, and secrets are handled.
- Exclusions: generated, vendored, dependency, cache, and build output, unless the user includes them. Lockfiles and deploy config stay in.
- Hotspots: files recurring in `git log -30 --name-only --format=`. Churn prioritises reading; it is not evidence.
- Safety net and blockers: tests, lint/format config, CI; missing deps, credentials, network, or runtimes that block verification.

### 3. Review in slices

Partition the in-scope code into slices by subsystem, each a cohesive area one reviewer can read thoroughly, so each file is read by one reviewer. Add a **repo-wide** slice for manifests, lockfiles, CI and deploy config, and cross-cutting auth or middleware. A repo small enough for one reviewer is reviewed inline with no sub-agents.

Dispatch one sub-agent per slice in parallel. Where the harness lets you choose, slice reviewers can run on a faster model; step 4 re-checks what matters most. Each brief carries the audit root, the map, its slice and exclusions, the bar, the passes, and step 4's verification rules verbatim, and ends: "Read entry points, trust boundaries, and hotspots first, then surrounding code and callers. Classify each candidate with the validate-finding skill and return only `bug` and `recommendation` findings that clear the bar, each with a one-line claim, `file:line`, the concrete failure or cost, evidence or a reproduction, and confidence. Also return coverage gaps and leads that cross into other slices."

Every reviewer applies all five passes:

- **Correctness**: edge cases, assumptions the system doesn't guarantee, invariants enforced at only some call sites.
- **Security**: untrusted input reaching a sink; missing authn/authz, especially object-level; secrets or PII, reported by location, type, and redacted fingerprint, never the value. A dependency CVE counts only after confirming the locked version and dependency path against an audit tool or current advisory.
- **Architecture**: layer leaks, coupling or cycles, drifting duplicated state, an invented pattern where one exists, compat breaks for callers or stored data.
- **Performance & reliability**, only where realistic volume makes it concrete: N+1, missing indexes, unbounded queries, heavy request-path work, memory growth; external calls without timeouts, unsafe retries, silent failures, overlapping or silently-stopped jobs, prod-only config breaks, non-atomic multi-store writes.
- **Maintenance**: EOL runtimes, deprecated APIs, confirmed-unmaintained deps, dead flags and unreachable code (check time-sensitive claims against current docs); behaviour-preserving simplifications with a concrete payoff. A documented repo convention suppresses the finding.

Done when every slice returns findings or an explicit none, plus coverage gaps.

### 4. Verify

Merge duplicates across slices and chase cross-slice leads. Reviewers have already classified their findings; re-check yourself only **Fix now** candidates, low-confidence findings, and claims spanning slices.

Where safe, confirm with a targeted test, build, static analysis, or minimal repro; record the command and result, and separate pre-existing failures from the suspected defect. Never run write-mode formatters. A security finding can instead rest on a complete source-to-sink trace when running the exploit is unsafe.

`bug` and `recommendation` findings that clear the bar survive. Drop false positives and anything tooling already enforces; a `possible issue` is a coverage gap, not a finding.

### 5. Report

One list, worst first. Tag each **Fix now** (bugs, security and data-loss issues, reliability failures) or **Should fix** (maintainability cost). Each: one-line claim, `file:line`, the failure or cost it causes, evidence, fix.

End with one line: counts per tag and false positives dropped. If verification was materially constrained, add one sentence on audit limitations. An empty list is a fine outcome; state it plainly.
