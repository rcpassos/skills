# Follow-up reviews

Read the prior review record and verify its target identity and referenced revisions before narrowing scope. Treat its decisions as context; current code and contracts determine whether they still hold.

## Establish the delta

Compare the previous reviewed head with the new pinned head using a two-endpoint diff. Retain the full current merge-base-to-head diff for context. Check whether the base, spec, or relevant standards changed too.

If history was rewritten, map the old and new changes using the recorded SHAs and surviving patches. When a dependable mapping is unavailable, perform an initial review and explain the scope expansion. A missing record likewise cannot support a claim that earlier findings were verified.

An unchanged head still permits rerunning previously blocked checks or settling missing facts. It does not justify a new full review by itself.

## Review fixes and affected behavior

Check every open finding and any resolved or rejected finding whose code, contract, or assumptions changed. For each claimed fix, trace the original failure and check the affected callers, state transitions, and shared invariants for regressions. A changed line or newly added test alone does not prove resolution.

Cover all new changes since the last review, including changes unrelated to the reported fixes. Use the Standards, Spec, and Risk axes where applicable to that delta; delegate independent scopes when useful. Expand to other code when a changed shared contract or concrete evidence warrants it, and record why. If a new bug emerges during this work, validate it even if it lies outside the original findings.

Reuse a finding's ID for the same root cause, including an incomplete fix. Reopen resolved or rejected findings only when new evidence or changed assumptions defeats the recorded reason; state what changed. Respect deferred decisions while keeping their impact and any gate exception visible.

## Close the pass

Apply the main skill's verification and completion rules. Record each finding's current status, newly demonstrated bugs, checks performed, scope expansions, and remaining gaps. If the same code and evidence leave the same unresolved finding, report it once as still open and name the needed action; repeating the search is not progress.

Return to the main skill to report and save the record. Finish after this scoped pass; optional improvements alone do not trigger another review cycle.
