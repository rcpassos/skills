# Doc templates

Fill every `<...>`; drop a section that does not apply rather than leaving it empty. `/wp-implement-rcp` reads these files and updates them as phases finish, so keep the headings and status values exactly as shown.

## `AGENTS.md`

The always-loaded entry point for implementation sessions. Keep it short: rules and pointers, not inventories.

```markdown
# AGENTS.md

<Site name>: WordPress build from Figma, implemented phase by phase. Run `/wp-implement-rcp` to implement the next phase of `docs/plan.md`.

## Targets

- Site: <URL> — <staging | production>. <Backup/rollback approach.>
- WordPress access: Novamira MCP server `<server name>`. Never update, deactivate, or delete the Novamira plugin or the connecting user's credentials.
- Design: Figma file <URL> via the Figma MCP.

## Build approach

<Page builder + mode / Gutenberg / coded theme>, chosen in `docs/decisions.md`. Use only this approach. Before building, load these Novamira skills with `novamira/skill-get`: <list>.

## Conventions

- Styles come from the globals in `docs/design-system.md`; add a token there before using a new value.
- <Naming: templates, CSS classes, field keys, page slugs.>
- <Site-specific rules, e.g. from Novamira memories: "native Elementor widgets only, no HTML widget".>
- Prefer a dedicated Novamira ability over `execute-php`; write PHP only into the child theme or the Novamira sandbox.
- Before overwriting existing content, save its current tree to `.scratch/backups/<post-id>-<date>.json`.

## Docs

- `docs/plan.md` — phases, status, handoff notes. Source of truth for what to do next.
- `docs/decisions.md` — settled decisions and why. Read before changing approach.
- `docs/site-map.md` — page → slug → Figma nodes → WordPress ID.
- `docs/design-system.md` — Figma variables → WordPress globals.
- `docs/content-model.md` — CPTs, taxonomies, field groups.
```

## `CLAUDE.md`

Only if absent; if present, ensure it imports `AGENTS.md`.

```markdown
@AGENTS.md
```

## `docs/plan.md`

```markdown
# Build plan: <site name>

Figma: <file URL> · Site: <URL> · Approach: <build approach>

Status values: `todo`, `in progress`, `done`, `blocked`. Implement phases in order; a phase starts only when every phase in its **Depends on** is `done`.

## Phase 1 — <title>

- **Status:** todo
- **Goal:** <one sentence: what exists on the site when this is done>
- **Depends on:** none
- **Figma:** <node URLs with node-id, per breakpoint>
- **WordPress targets:** <pages, templates, globals, field groups — by slug or name>
- **Novamira skills:** <skills to load for this phase>

### Tasks

1. <Concrete action>

### Acceptance criteria

- [ ] <Observable check, e.g. "/about/ at 1440 and 390 matches frames <node>, <node>">

### Out of scope

- <What a session might be tempted to do here but belongs to a later phase>

### Handoff

<Empty until the phase finishes. Then: what was built, WordPress IDs created, deviations from the design and why, follow-ups.>
```

Repeat the phase block for each phase.

## `docs/decisions.md`

One entry per decision from planning; implementation sessions append.

```markdown
# Decisions

## <Decision title>

- **Decision:** <what was chosen>
- **Why:** <the reason, including rejected options>
- **Date / by:** <YYYY-MM-DD> / <user or phase>
```

## `docs/site-map.md`

```markdown
# Site map

| Page | Slug | Figma (desktop / tablet / mobile) | Template / parts | Phase | WP ID |
| --- | --- | --- | --- | --- | --- |
| <Home> | `/` | <node URLs> | <header, footer> | <n> | <filled when built> |

## Global parts

| Part | Figma | Applies to | Phase | WP ID |
| --- | --- | --- | --- | --- |
```

## `docs/design-system.md`

```markdown
# Design system

Source: Figma variables from <file URL>. WordPress location: <Elementor kit globals | theme.json | child theme CSS custom properties | Novamira design slug>.

## Colors

| Figma variable | Value | WordPress global | Notes |
| --- | --- | --- | --- |

## Typography

| Figma style | Family / weight / size / line height | WordPress global |
| --- | --- | --- |

## Spacing, radii, breakpoints

<Scale and the breakpoints verified, mapped to the builder's breakpoints.>

## Raw values in the design

<Values used in Figma without a variable, and the token each maps to.>
```

## `docs/content-model.md`

Only when the plan has CPTs, taxonomies, field groups, or options pages.

```markdown
# Content model

Owner plugin: <ACF Pro | ...> — create through it, never in custom PHP.

## <Post type or options page>

- **Slug / key:** <...>
- **Shown in:** <templates, pages>
- **Fields:** <name — type — required — notes>
- **Taxonomies:** <...>
- **Translation:** <translate | copy | ignore>
```
