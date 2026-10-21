---
name: audit-rcp
description: Audit a local codebase or GitHub repo for bugs, security, architecture, performance, reliability, and maintenance costs; carry findings and decisions into scoped follow-up checks of fixes. For PR or diff reviews, use a dedicated change-review skill.
disable-model-invocation: true
---

Initial audits run five independent full-project passes, then verify findings. Follow-ups check earlier findings, fixes, and affected behavior. Leave implementation unchanged unless the user asks for fixes; save only the audit record by default. The user's instructions override this workflow.

## The bar

A finding has a concrete cost: a bug, a security or data-loss issue, a reliability or performance failure at realistic load, or a maintainability cost — code that makes future change harder or riskier (duplicated logic that will drift, coupling across layers, deprecated or EOL dependencies, a risky path without tests, a simplification with a concrete payoff). Style, nits, YAGNI, and taste fall below it; below-bar findings go unreported and uncounted.

## Process

### 1. Resolve the target and choose the mode

- **Local path** — confirm it resolves; that directory is the audit root.
- **GitHub reference** (URL, `owner/repo`) — `git clone --depth 31 <url>` into a `mktemp -d` directory (31 commits feed the hotspot window) and record `git rev-parse HEAD`. Never push or touch the remote.
- Ambiguous between local and remote — ask.

Find prior findings and decisions in the current conversation, a user-supplied record, or `.scratch/audit/`. Match the repository and audited scope, not just a filename. **Initial audit** covers the full project. **Follow-up** applies when the user asks whether earlier findings are fixed or continues that audit; read [references/follow-up.md](references/follow-up.md). An explicit fresh audit runs the full passes while retaining earlier decisions as context.

Identify the current revision and working-tree changes; use a stable snapshot when changes are uncommitted, or record a content snapshot for a non-Git directory. Read and verify against that same snapshot. Done when the target, snapshot, mode, and covered scope are established.

### 2. Build the grounding map

For an initial audit, skim once, then hand the map to every pass. For a follow-up, refresh the parts needed for its scoped checks:

- What the project claims to be and its conventions: `README.md`, `CONTEXT.md`, `AGENTS.md` / `CLAUDE.md`, `docs/adr/`.
- Runtime and exact dependency versions from manifests and lockfiles.
- Layout: entry points, layer boundaries, where auth, input, and secrets are handled.
- Exclusions: generated, vendored, dependency, cache, and build output — unless the user includes them. Lockfiles and deploy config stay in.
- Hotspots: files recurring in `git log -30 --name-only --format=`. Churn prioritises reading; it is not evidence.
- Safety net: tests, lint/format config, CI.
- Constraints that block verification: missing deps, credentials, network, runtimes.

### 3. Run the scoped checks

For an initial audit, dispatch one sub-agent per pass, in parallel as capacity allows. For a follow-up, use the reference above to select scoped checks instead of rerunning all five passes. Each delegated check gets the audit root, snapshot, map, exclusions, prior decisions, assigned scope, relevant checklist, and the bar verbatim, and is told: "Report only evidence-backed leads that clear the bar — each with a one-line claim, `file:line`, the concrete failure, and evidence or a reproduction. Read the surrounding code and callers before reporting."

- **Correctness** — hidden bugs the happy path and tests miss; edge cases (empty/huge input, falsy values, boundaries, unicode, timezones, repeat/concurrent calls, partial failure); assumptions the system doesn't guarantee (nullability, ordering, uniqueness, freshness, unchecked casts, invariants enforced at only some call sites).
- **Security** — untrusted input reaching a query, command, path, template, or deserializer; missing authn/authz, especially object-level; secrets or PII in code, logs, or responses; weak crypto or randomness, open redirects, missing rate limits. Report a secret by location, type, and redacted fingerprint — never its value. A dependency CVE counts only after confirming the locked version and dependency path against an audit tool or current advisory.
- **Architecture** — code that fights the codebase's shape (layer leaks, new coupling or cycles, drifting duplicated state, an invented pattern where one exists); compat breaks for callers or stored data.
- **Performance & reliability** — N+1, missing indexes, unbounded queries, heavy request-path work, memory growth — only where realistic volume makes it concrete; external calls without timeouts, unsafe retries, silent failures, overlapping or silently-stopped jobs, prod-only config breaks, non-atomic multi-store writes.
- **Maintenance** — EOL runtimes, deprecated APIs, confirmed-unmaintained deps, dead flags and unreachable code (verify time-sensitive claims against current docs); behaviour-preserving simplifications with a concrete payoff. A documented repo convention suppresses the finding.

### 4. Verify every finding

Pool the scoped checks or passes, merge duplicates by root cause, and classify each against the snapshot with [validate-finding](../validate-finding/SKILL.md). Where safe, confirm with a targeted test, build, static analysis, or minimal repro — record the command and result, and separate pre-existing failures from the suspected defect. Never run write-mode formatters. A security finding can instead be proven by a complete source-to-sink trace when running the exploit is unsafe.

`bug` and `recommendation` survive. Drop false positives, anything tooling already enforces, and anything below the bar. A `possible issue` is not a survivor.

### 5. Report

One list, worst first. Tag each **Fix now** (bugs, security and data-loss issues, reliability failures) or **Should fix** (maintainability cost). Each: stable ID (`A1`, `A2`, ...), one-line claim, `file:line`, the failure or cost it causes, evidence, fix. Follow-ups report status changes and new findings without repeating unchanged explanations; state whether the earlier findings are resolved, still open, deferred, rejected, or unverified.

End with counts of open findings per tag and false positives dropped, plus any material coverage or verification limits. An empty list is a fine outcome — state it plainly. Finish after the scoped checks and verification; another broad search requires an explicit fresh audit or concrete evidence warranting a recorded scope expansion. A completed follow-up describes the checked scope and snapshot, not proof of a defect-free project.

### 6. Save the audit record

Update the matching record, or create `.scratch/audit/<target>.md` using a filesystem-safe repository or scope identifier. Link it in the final report; keep it out of commits unless requested. If the user requires no writes, provide the record inline instead.

Keep a compact current record containing:

- Repository and scope identity, snapshot or revision identity (including uncommitted changes), mode, exclusions, contracts, and relevant standards.
- Stable finding IDs, root cause, classification, tag, location, evidence, and status: open, resolved, rejected, or deferred. Record the reason and who made any acceptance or deferral decision; assign new IDs only to new root causes.
- Checks and results, scope expansions, verification gaps, and the snapshot against which each resolution was verified. An unverified claim stays open with the missing evidence named.

Preserve enough resolved and rejected evidence to avoid reopening settled claims without cause; omit discarded nits. Done when the record matches the reported snapshot, statuses, and decisions.
