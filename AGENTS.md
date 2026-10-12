# AGENTS.md

Personal agent skills, shared by Claude Code and Codex. No build, no tests — a skill is verified by running it.

## Layout

Each skill lives in `skills/<category>/<name>/` (`general`, `wordpress`, …); a new category is a new folder plus a matching section in `README.md`. A skill holds:

- `SKILL.md` — frontmatter (`name`, `description`) plus the body the agent runs.
- `agents/openai.yaml` — Codex metadata: `display_name`, `short_description`, optional `default_prompt`, `policy`.

## Conventions

- Suffix every user-facing skill `-rcp`; the directory name, `name:` frontmatter, and `display_name` (`"<Title> (RCP)"`) stay in lockstep.
- User-facing skills are user-invoked only. Set both switches together: `disable-model-invocation: true` in `SKILL.md` and `policy.allow_implicit_invocation: false` in `openai.yaml`.
- Helper skills (`validate-finding`) are loaded by other skills, so they stay model-invocable and unsuffixed.
- Skills depend only on skills in this repo — installing `rcpassos/skills` must bring everything a skill references.
- Link another skill as `../<name>/SKILL.md`. Installs are flat (category folders dropped), so that path resolves both installed and here only when both skills share a category.
- Before writing or editing a `SKILL.md` or this file, load the `writing-for-agents` skill.

## Installation gotcha

The installed skills in `~/.agents/skills/<name>` are **copies**, not symlinks to this repo (`~/.claude/skills/<name>` symlinks to those copies). An edit here reaches the agents only after it is copied over — check with `diff -r skills/<category>/<name> ~/.agents/skills/<name>`.
