---
name: audit-rcp
description: Audit a local codebase or GitHub repo for hidden bugs, edge cases, incorrect assumptions, security holes, architectural problems, performance and reliability risks, deprecated code, and simplifiable code. Use when the user asks to audit, health-check, review a full project, harden a repo, or find tech debt. Do not use for reviewing changes in a PR, diff, branch, or commit; use a dedicated change-review skill instead.
disable-model-invocation: true
---

Read-only full-project audit, not a diff review: five independent passes, then verify and triage into two lists. Stay read-only unless the user asks for fixes; their instructions override this workflow.

## Process

### 1. Resolve the target

- **Local path** — confirm it resolves; that directory is the audit root.
- **GitHub reference** (URL, `owner/repo`) — `git clone --depth 31 <url>` into a `mktemp -d` directory (31 commits feed the hotspot window) and record `git rev-parse HEAD`. Never push or touch the remote.
- Ambiguous between local and remote — ask.

### 2. Build the grounding map

Skim once, then hand the map to every pass:

- What the project claims to be and its conventions: `README.md`, `CONTEXT.md`, `AGENTS.md` / `CLAUDE.md`, `docs/adr/`.
- Runtime and exact dependency versions from manifests and lockfiles.
- Layout: entry points, layer boundaries, where auth, input, and secrets are handled.
- Exclusions: generated, vendored, dependency, cache, and build output — unless the user includes them. Lockfiles and deploy config stay in.
- Hotspots: files recurring in `git log -30 --name-only --format=`. Churn prioritises reading; it is not evidence.
- Safety net: tests, lint/format config, CI.
- Constraints that block verification: missing deps, credentials, network, runtimes.

### 3. Run the five passes

One sub-agent per pass, in parallel as capacity allows. Each gets the audit root, the map, the exclusions, and its checklist verbatim, and is told: "Report only your strongest evidence-backed leads — each with a one-line claim, `file:line`, the concrete failure, and evidence or a reproduction. Read the surrounding code and callers before reporting."

- **Correctness** — hidden bugs the happy path and tests miss; edge cases (empty/huge input, falsy values, boundaries, unicode, timezones, repeat/concurrent calls, partial failure); assumptions the system doesn't guarantee (nullability, ordering, uniqueness, freshness, unchecked casts, invariants enforced at only some call sites).
- **Security** — untrusted input reaching a query, command, path, template, or deserializer; missing authn/authz, especially object-level; secrets or PII in code, logs, or responses; weak crypto or randomness, open redirects, missing rate limits. Report a secret by location, type, and redacted fingerprint — never its value. A dependency CVE counts only after confirming the locked version and dependency path against an audit tool or current advisory.
- **Architecture** — code that fights the codebase's shape (layer leaks, new coupling or cycles, drifting duplicated state, an invented pattern where one exists); compat breaks for callers or stored data.
- **Performance & reliability** — N+1, missing indexes, unbounded queries, heavy request-path work, memory growth — only where realistic volume makes it concrete; external calls without timeouts, unsafe retries, silent failures, overlapping or silently-stopped jobs, prod-only config breaks, non-atomic multi-store writes.
- **Maintenance** — EOL runtimes, deprecated APIs, confirmed-unmaintained deps, dead flags and unreachable code (verify time-sensitive claims against current docs); behaviour-preserving simplifications with a concrete payoff. A documented repo convention suppresses the finding.

### 4. Verify every finding

Pool the passes, merge duplicates, and classify each against the actual code with the `validate-finding` skill. Where safe, confirm with a targeted test, build, static analysis, or minimal repro — record the command and result, and separate pre-existing failures from the suspected defect. Never run write-mode formatters. A security finding can instead be proven by a complete source-to-sink trace when running the exploit is unsafe.

`bug` and `recommendation` survive. Drop false positives, anything tooling already enforces, and taste with no defensible cost. A `possible issue` is not a survivor.

### 5. Report

**Take it** — survivors worth fixing now, worst first. Each: one-line claim, `file:line`, the failure it causes, evidence, fix.

**Leave it** — minor, YAGNI, or low-payoff. One line each with why it can wait.

End with one line: counts per list and false positives dropped. If verification was materially constrained, add one sentence on audit limitations. An empty **Take it** is a fine outcome — state it plainly.
