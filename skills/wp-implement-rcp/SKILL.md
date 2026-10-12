---
name: wp-implement-rcp
description: Implement one phase of a WordPress build plan (docs/plan.md from /wp-plan-rcp) from Figma via the Figma and Novamira MCPs, verify it, and hand off to the next session.
disable-model-invocation: true
---

Implement exactly one **phase** of `docs/plan.md` on the WordPress site, verify every acceptance criterion against the site and the Figma frames, and leave a **handoff** the next session can start from with no other context. The plan is the contract: build what the phase lists, nothing from later phases.

## Process

### 1. Load the phase

Read `AGENTS.md`, `docs/plan.md`, and `docs/decisions.md`. Pick the phase:

- The one the user names.
- Otherwise a phase marked `in progress` — resume it from its handoff notes and the site's current state.
- Otherwise the first `todo` phase whose dependencies are all `done`.

A dependency not `done`, or no plan → stop and say so (`/wp-plan-rcp` creates the plan). Confirm the Figma MCP (`whoami`) and the Novamira server named in `AGENTS.md` both answer, and that the site matches the one in `AGENTS.md`. Done when the phase is chosen and both servers answer.

### 2. Gather context

- **Site rules** — the Novamira instructions from `discover-abilities`, its memories, and the Novamira skills the phase and `AGENTS.md` name (load each with `novamira/skill-get` before building).
- **Design** — for every Figma node in the phase: `get_design_context` and `get_screenshot`, per breakpoint; `get_variable_defs` where the phase touches tokens. On a large frame, `get_metadata` first and fetch per section.
- **Current state** — read every WordPress target (`elementor-get-content`, `gutenberg-get-content`, `acf-*` reads, template lists). Back up any target with content to `.scratch/backups/<post-id>-<date>.json` before the first write.
- **Lookups** — `docs/design-system.md` for globals, `docs/site-map.md` for slugs and IDs, `docs/content-model.md` for fields.

Set the phase status to `in progress` in `docs/plan.md`. Done when every node in the phase has design context and a screenshot, and every target's current state is known.

### 3. Build

Work the tasks in order, with the build approach from `AGENTS.md`:

- Bind styles to the globals in `docs/design-system.md`. A value with no global → add the token (globals and the doc) when the phase owns foundations; otherwise record it as a follow-up and use the nearest global.
- Use dedicated Novamira abilities; reach for `execute-php` only where none exists, and keep written PHP in the child theme or the Novamira sandbox.
- Content the design shows as repeated or editable goes through the content model, not hardcoded copy.
- Discoveries outside the phase (bugs, missing designs, better ideas) go into the phase handoff as follow-ups.

When the plan conflicts with the design or the site — a node is missing, a decision no longer fits, a task is impossible — stop and ask; record the answer in `docs/decisions.md`. Done when every task is done or explicitly blocked.

### 4. Verify

Check every acceptance criterion against the live site:

- **Visual** — render each page or template at each breakpoint in the plan (a browser tool; `novamira/create-admin-access-link` for drafts and logged-in views) and compare with the Figma screenshot: layout, spacing, typography, color, images, states. Fix differences and re-check. When no browser tool is available, list the URLs and breakpoints for the user to check and leave those criteria unchecked.
- **Design rules** — `novamira/check-design` when a Novamira design is active.
- **Function** — links, menus, forms, dynamic fields resolve, other languages render, no PHP warnings on the page.

A criterion is checked only when observed to pass. Done when every criterion is checked, or unchecked with the reason and what would fix it.

### 5. Hand off

Update the docs so the next session needs nothing from this one:

- `docs/plan.md` — tick passed criteria; status `done` when all pass, otherwise `blocked` or `in progress` with the reason. Fill **Handoff**: what was built, WordPress IDs created, deviations from the design and why, follow-ups.
- `docs/site-map.md`, `docs/design-system.md`, `docs/content-model.md` — real WordPress IDs, globals, and field keys created this phase.
- `docs/decisions.md` — decisions made during the phase.
- New site conventions learned the hard way → `AGENTS.md`.

Leave changes uncommitted unless the user asks. Done when the docs match the site.

### 6. Report

Phase and status, criteria passed/failed, deviations, follow-ups, and the next phase to run in a new session.
