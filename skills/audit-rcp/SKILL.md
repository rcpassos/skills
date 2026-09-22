---
name: audit-rcp
description: Audit a local codebase or GitHub repo for hidden bugs, edge cases, incorrect assumptions, security holes, architectural problems, performance and reliability risks, deprecated code, and simplifiable code. Use when the user asks to audit, health-check, review a full project, harden a repo, or find tech debt. Do not use for reviewing changes in a PR, diff, branch, or commit; use a dedicated change-review skill instead.
disable-model-invocation: true
---

Perform a read-only full-project audit, not a diff review. Cover nine axes grouped into five passes: Correctness, Security, Architecture, Performance & Reliability, Maintenance. Run independent passes, then verify and triage their findings. Keep the audit read-only unless the user explicitly asks for fixes. The user's instructions take precedence over this workflow.

## Process

### 1. Resolve the target

The user gives either a local path or a GitHub reference (URL, `owner/repo`, or issue-linked repo).

- Local path: confirm it resolves and list top-level entries. That directory is the audit root. Do not modify it.
- GitHub: create an isolated temporary directory with `mktemp -d`, then clone into it with enough history for the 30-commit hotspot window (`git clone --depth 31 <url> <temp-dir>`). Record the full commit SHA with `git rev-parse HEAD`. Never push, open a PR, or modify the remote.
- If the target is ambiguous, ask. Do not guess between local and remote.

### 2. Ground before judging

Skim before spawning sub-agents (you do this once, then hand the map to them):

- Entry docs: `README.md`, `CONTEXT.md`, `AGENTS.md` / `CLAUDE.md`, `docs/adr/` — what the project says it is and the conventions it claims.
- Manifests and lockfiles: `package.json`, `go.mod`, `Cargo.toml`, `pyproject.toml`, `Gemfile`, `*.csproj`, and ecosystem lockfiles — runtime version, exact dependency versions, and available scripts.
- Layout: entry points, transport/UI vs domain vs data layers, where auth/input/secret handling lives.
- Inventory and exclusions: identify first-party source and configuration. Exclude generated files, vendored code, dependency directories, caches, and build output unless the user explicitly includes them. Still inspect lockfiles and deployment configuration.
- Hot spots: for a repo with sufficient history, use `git log -30 --name-only --format=` and aggregate repeatedly changed files. Use churn only to prioritize inspection; it is not evidence of a defect.
- Existing safety net: test dirs, lint/format config, CI workflow. Treat missing coverage as an audit limitation unless it creates a concrete, evidenced risk.
- Audit constraints: record unavailable dependencies, missing credentials, disabled network access, unsupported runtimes, or other limits that prevent verification.

Pass the audit root + this map to every sub-agent. Each sub-agent reads the surrounding file and callers of anything it flags — findings that only show up outside the flagged hunk are the norm.

### 3. Run the five independent passes

Use the available collaboration tools and respect their current concurrency limit. Run passes in parallel when capacity allows; otherwise assign one pass to the primary agent or queue the remaining pass. Do not invent an agent or tool type that the runtime does not provide.

Give each worker the audit root, grounding map, exclusions, and its checklist verbatim. Brief each: "Report only the strongest evidence-backed leads. For each: one-line claim, `file:line`, concrete failure, and supporting evidence or reproduction. Read surrounding code and callers before reporting it."

**Correctness**

- **Hidden bugs** — logic wrong in a way tests and happy path don't reveal: off-by-one, inverted condition, wrong variable, mutation of shared state, unawaited promise, swallowed error, resource never released, race between concurrent paths.
- **Edge cases** — empty, single-element, very large inputs; `null`/`undefined`/`NaN`/`0`/`""` where truthiness is tested; boundary values; duplicates; unicode; timezones and DST; concurrent or repeated calls (idempotent?); partial failure and retry; network timeout; pagination limits.
- **Incorrect assumptions** — what code takes for granted the system doesn't guarantee: optional/nullable field treated as present, ordering not guaranteed, uniqueness not enforced, call that can fail or return partial data, cache/clock assumed fresh, type or non-null assertion over unverified shape, invariant enforced at one call site but not others.

**Security**

- Untrusted input reaching query, command, path, template, or deserializer without validation (injection, path traversal, SSRF).
- Missing/wrong authn/authz on endpoint, handler, or resource — especially object-level checks (can user A pass user B's id?).
- Secrets, tokens, PII in code, logs, error messages, client-visible responses. Never reproduce a secret or personal value in output; report only its location, type, and a redacted fingerprint when needed to distinguish occurrences.
- Weak or hand-rolled crypto, security-purpose randomness, unsafe redirects, missing rate limiting on expensive/auth-adjacent paths.
- Dependencies: known-vulnerable, deprecated, or unmaintained packages. Treat unfamiliar packages only as investigation leads. Before reporting, establish the installed or locked version and dependency path, then confirm the claim with an ecosystem audit tool or a current authoritative advisory. If current evidence is unavailable, do not present the claim as a finding.

**Architecture**

- Business logic leaking into transport/UI layer, inner layer importing outer one, new cross-module coupling or cycle, state duplicated in two places that can drift, leaky abstraction, boundary the codebase keeps closed being crossed, invented pattern when repo already has one.
- Migration, backward-compat, data-shape breaks for existing callers or stored data.

**Performance & Reliability**

- **Performance** — N+1 queries, missing indexes on filtered/joined/sorted columns, unbounded queries or result sets, slow loops over large data, heavy work on the request path that belongs in a background job, missing or incorrect caching, memory growth (leaks, loading whole files or tables into memory). Report only what realistic data volume or traffic makes concrete: cite the query, loop, or path, and measure when you can.
- **Reliability & operations** — external calls without timeouts; retries that are missing, unbounded, or not idempotent; errors swallowed without logging; logs missing the context needed to debug (or leaking secrets/PII); queue, cron, and scheduler jobs that can overlap, double-run, or stop silently; environment-specific configuration that breaks in production; writes across multiple stores with no consistency guarantee.

**Maintenance — deprecated + simplifiable**

- **Deprecated code** — EOL runtime in manifests/Docker/CI; deprecated stdlib/framework APIs; dependencies confirmed unmaintained or covered by a published advisory; pinned versions far behind upstream with a concrete migration cost; dead feature flags, commented-out blocks, unreachable code guarding removed callers. Verify time-sensitive claims against current authoritative documentation.
- **Code improvements** — code that can be simplified without behaviour change: duplicated logic shape (extract and call twice); mysterious name hiding intent (rename, or design is murky); primitive standing in for a domain concept (small type); repeated switch on same type (polymorphism or shared map); middle-man that only delegates (cut it); speculative generality with no caller (delete it). Repo convention overrides — where docs endorse it, suppress the finding.

### 4. Double-check every finding

Pool all five passes, then confirm each against the actual code, classifying it with the `validate-finding` skill's rubric and evidence standards: `bug` and `recommendation` can survive (step 5 sorts them); `false-positive` is dropped; `possible issue` is an unverified lead. When safe and relevant, run a targeted test, existing test/build/static-analysis command, or minimal reproduction. Do not use write-mode formatters or mutate the target. Record the command and result used for confirmation, and distinguish pre-existing suite failures from failures caused by the suspected defect. A security finding may instead be verified through a complete, concrete source-to-sink analysis when executing an exploit would be unsafe.

Drop anything where:

- the code doesn't do what the finding claims (misread, missing guard/validation/type/test elsewhere);
- tooling already enforces or auto-fixes it (formatter, linter, compiler);
- it's a restatement of another finding — merge duplicates into one;
- it's taste with no defensible cost.

An unverified lead is not a survivor. Do not place it in **Take it** or **Leave it**. Mention it only as an audit limitation after the verdict counts when the missing verification materially affects confidence.

### 5. Report two lists

Present only survivors, in exactly two sections:

**Take it** — verified bugs, correctness, security, data-loss, breakers, architectural problems worth fixing now, performance or reliability problems with concrete impact, deprecated code with confirmed EOL/CVE/migration cost, and simplifications with concrete payoff. Order worst first. Each: one-line claim, `file:line`, concrete failure it causes, evidence, and fix.

**Leave it** — minor improvements, style nits, YAGNI, harmless dead code, low-payoff simplifications. One line each, plus why not worth doing now.

End with one line: counts per list, plus how many dropped as false positives. If verification was materially constrained, append one concise audit-limitations sentence without turning unverified leads into findings.

Do not pad either list. An empty **Take it** is a fine outcome — state it plainly.
