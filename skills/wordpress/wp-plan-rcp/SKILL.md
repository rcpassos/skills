---
name: wp-plan-rcp
description: Plan a WordPress site build from Figma designs via the Figma and Novamira MCPs — clarifying questions, a phased docs/plan.md, and the agent docs each implementation session needs.
disable-model-invocation: true
---

Turn a Figma design and a Novamira-connected WordPress site into a **phased plan**: `docs/plan.md` plus the agent docs that let a fresh session implement one phase with `/wp-implement-rcp` and nothing else. Survey both sides, ask until every decision is settled, then write. Planning is read-only on Figma and WordPress; the only writes are files in this repo.

A **phase** is one session of work: one goal, explicit dependencies, the exact Figma nodes and WordPress targets it touches, and acceptance criteria a fresh agent can check without asking anyone.

## Process

### 1. Preflight

- **Figma MCP** — call `whoami`. Get the Figma file URL(s) from the user if not given; note the file key and keep every node as a full URL with `node-id`.
- **Novamira MCP** — call `discover-abilities` (or the `novamira/agent-context` ability) and keep its environment block and instructions; they are the site's rules.
- **Existing plan** — if `docs/plan.md` exists, ask whether to revise it (keep finished phases) or replace it.

Missing MCP → stop and tell the user which server to connect. Done when both servers answer and the Figma file is known.

### 2. Survey the site

Read only — use read abilities and read-only queries through `novamira/execute-php`. Record:

- WordPress, PHP, theme (parent/child), active plugins, languages (WPML/Polylang).
- Page builder and its mode: `elementor-check-setup` (v4 atomic vs v3), Gutenberg, or coded theme.
- Existing pages, menus, templates/theme-builder parts, CPTs, taxonomies, field groups (`acf-*` list abilities), forms, SEO setup (`yoast-check-setup`).
- Existing design state: `list-design-library`, `get-active-design`, Elementor globals (`elementor-list-v3-styles`).
- Novamira memories (`memory-list`) and the Novamira skills listed in the instructions — note which ones the build approach will need (e.g. `elementor-build-page`, `gutenberg-edit-content`, `custom-theme-build`, `novamira-design`, `acf-integration`, `dynamic-data-binding`, `yoast-integration`).
- Whether this is production. Novamira is meant for dev/staging; a production target becomes a decision in step 4.

Done when the inventory covers every item above.

### 3. Survey the design

- `get_metadata` on each page of the file to list top-level frames. Classify every frame: **page** (and which breakpoint), **template** (single CPT, archive, 404), **global part** (header, footer, menus), **component/section** reused across pages, **state** (hover, open menu, form error), or **ignore** (drafts, explorations).
- `get_variable_defs` for colors, typography, spacing, radii; note where the design uses raw values instead of variables.
- `get_screenshot` of each page frame to read intent the metadata misses.
- Spot repeated content (cards, team members, listings) — candidates for a CPT or field group rather than hand-built sections.

Large frames can overflow `get_design_context`; use `get_metadata` to find child node IDs and work per section. Done when every top-level frame has a classification and every page frame maps to a slug.

### 4. Clarify

Ask until each decision below is answered or the user accepts a stated assumption. Batch questions, lead each with the default you recommend and why, and ask only what the survey could not settle.

- **Build approach** — page builder (which, and v4 atomic vs v3 for Elementor), Gutenberg, or coded child/custom theme. Novamira requires one approach, chosen once.
- **Environment** — target site is staging or production; backup and rollback approach; who reviews.
- **Page inventory** — confirm the frame→page mapping; pages with no design; pages missing a breakpoint (and how to derive it).
- **Content model** — which repeated content becomes CPTs, taxonomies, ACF field groups, options pages; who edits what after launch.
- **Existing content** — keep, replace, or migrate what is already on the site.
- **Languages** — which languages, what gets translated, who supplies translations.
- **Behavior not in Figma** — forms and where submissions go, integrations, animations, search, filters, menus on mobile.
- **SEO** — titles/descriptions source, redirects from an old site, indexing.
- **Fidelity bar** — breakpoints to verify and what counts as a match.
- **Out of scope** — what this plan will not do.

Done when no decision above is open.

### 5. Draft the phases

Order by dependency: foundations (design tokens into the builder's globals or `theme.json`/CSS, fonts, Novamira design), content model, global parts, reusable sections, pages (one or a few per phase), dynamic data, multilingual, SEO, final QA. Split any phase that cannot finish in one session or whose acceptance criteria exceed roughly ten checks.

Every phase names exact Figma node URLs and WordPress targets, and has acceptance criteria that are observable on the site: a URL renders at each breakpoint and matches the frame screenshot, a field group exists with listed fields, a template applies to listed conditions. Show the user the phase outline (titles, goals, dependencies) and adjust before writing. Done when the user accepts the outline.

### 6. Write the docs

Write the files in [references/templates.md](references/templates.md), filled from the survey and answers. Merge into an existing `AGENTS.md`/`CLAUDE.md` rather than overwriting. Every fact lives in one file; the others link to it. Leave the files uncommitted unless the user asks.

Done when a fresh agent with only this repo and the two MCPs could start Phase 1: every phase has nodes, targets, and criteria; every decision from step 4 is in `docs/decisions.md`; no placeholder text remains.

### 7. Report

The phase list (one line each), assumptions the user accepted, and the next step: run `/wp-implement-rcp` in a new session, one phase per session.
