# Follow-up reviews

Verify the prior record's target identity and revisions before narrowing scope. Its decisions are context; current code and contracts decide whether they still hold.

## Establish the delta

Diff the previously reviewed head against the new pinned head; keep the full merge-base-to-head diff for context. Check whether the base, spec, or standards changed too.

If history was rewritten, map old to new changes through the recorded SHAs and surviving patches; without a dependable mapping, do an initial review and say why. An unchanged head still allows rerunning blocked checks or settling missing facts, not a new full review.

## Review fixes and affected behaviour

Check every open finding, plus any resolved or rejected finding whose code, contract, or assumptions changed. For each claimed fix, trace the original failure through the affected callers, state transitions, and shared invariants; a changed line or new test alone doesn't prove resolution.

Review all new changes since the last pass, related to the fixes or not, per step 3 of the main skill (inline or sliced by size). Expand beyond the delta only when a changed shared contract or concrete evidence warrants it, and record why.

Reuse a finding's ID for the same root cause, including an incomplete fix. Reopen a resolved or rejected finding only when new evidence defeats the recorded reason; say what changed. Deferred decisions stand, with their impact and any gate exception visible.

## Close the pass

Return to steps 4–6 of the main skill. A finding left unchanged by the code and evidence is reported once as still open, with the action it needs. Optional improvements alone don't trigger another cycle.
