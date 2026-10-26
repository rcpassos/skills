# Follow-up audits

## Recover the earlier audit

Use the current conversation, a user-supplied record, or a matching record under `.scratch/audit/` to recover the target, findings, decisions, and last audited snapshot. Conversation history is sufficient for a same-session follow-up; reconstruct a compact record when needed.

If earlier findings are unavailable, ask for that report rather than substituting a new full audit. If only the old snapshot is unavailable, verify each recoverable finding against the current code and state which change or regression checks remain inconclusive. A missing record or rewritten history alone does not authorize a full audit.

Done when each earlier finding has an identity, claim, prior status, and supporting evidence, or the missing context is explicit.

## Check fixes and affected behavior

Check every open finding and any resolved or rejected finding whose code, contract, or assumptions changed. Trace the original failure through the current code, including affected callers and shared invariants; a changed line or new test alone does not prove resolution.

When snapshots are available, compare the last audited snapshot with the current one, including uncommitted changes. Use that delta to identify fixes, regressions, and affected paths. By default, check earlier findings, their fixes, and affected behavior; follow the user's requested scope if they also ask to review all intervening changes. An unchanged snapshot still permits verifying findings or rerunning blocked checks.

Run the scoped checks per step 3 of the main skill (inline, or sliced when they span several subsystems). Expand beyond the scope only for a changed shared contract, concrete evidence of a related failure, or a user request, and record why. Validate a concrete incidental discovery and report it separately from completion of the requested fix checks; it does not trigger another project-wide search.

Reuse the same ID for the same root cause, including incomplete fixes. Reopen resolved or rejected claims only when new evidence or changed assumptions defeats the recorded reason; state what changed. Respect deferrals while keeping their unresolved impact visible. Done when each checked finding has a current disposition and evidence or a named verification gap.

## Close the pass

Return to the main skill to report and save the record. Distinguish resolved findings from open, deferred, rejected, and unverified claims; verification gaps prevent claiming that all fixes are confirmed. State any unreviewed changes outside this follow-up's scope.

Finish after this scoped pass. If the same code and evidence leave the same finding unresolved, report it once as still open and name the needed action. Optional improvements and unchanged findings do not trigger another audit cycle.
