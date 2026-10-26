# skills

Agent skills for Claude Code, Codex, and other [skills](https://skills.sh)-compatible agents. The `-rcp` skills are user-invoked — run them by name (e.g. `/audit-rcp`); the agent won't trigger them on its own.

Skills are grouped by category under `skills/<category>/<name>/`.

### General

| Skill | What it does |
| --- | --- |
| [`audit-rcp`](skills/general/audit-rcp/SKILL.md) | Full-project audit of a local path or GitHub repo: verified findings, saved decisions, and scoped follow-up checks of fixes. Implementation stays unchanged. |
| [`code-review-rcp`](skills/general/code-review-rcp/SKILL.md) | Verified review of a PR, branch, or diff across Standards, Spec, and Risk, with severity, saved decisions, and follow-up checks of fixes. |
| [`docs-review-rcp`](skills/general/docs-review-rcp/SKILL.md) | Read-only documentation review: requirements, architecture, decisions, technical claims, Word fidelity, and diagrams. |
| [`issue-check-rcp`](skills/general/issue-check-rcp/SKILL.md) | Validate a GitHub issue before work starts: is it valid, is it useful, what's the risk, how big is it. |
| [`validate-finding`](skills/general/validate-finding/SKILL.md) | Classify a suspected issue as bug, possible issue, false-positive, or recommendation, backed by evidence. Used by `audit-rcp`, `code-review-rcp`, and `docs-review-rcp` to verify findings; also usable on its own. |

### WordPress

| Skill | What it does |
| --- | --- |
| [`wp-plan-rcp`](skills/wordpress/wp-plan-rcp/SKILL.md) | Plan a WordPress site build from Figma via the Figma and Novamira MCPs: clarifying questions, a phased `docs/plan.md`, and `AGENTS.md` plus supporting docs for implementers. |
| [`wp-implement-rcp`](skills/wordpress/wp-implement-rcp/SKILL.md) | Implement one phase of a `wp-plan-rcp` plan on the site, verify it against Figma, and write the handoff for the next session. |

### Laravel

| Skill | What it does |
| --- | --- |
| [`laravel-setup-rcp`](skills/laravel/laravel-setup-rcp/SKILL.md) | Configure a fresh `laravel new` project to house standards: Filament backoffice, Pint/Rector/PHPStan/Pest, Boost with house rules, Sanctum and Scramble for APIs. |

## Install

```bash
npx skills add rcpassos/skills
```

Install one skill with `--skill <name>`; list them first with `--list`; install for your user instead of the project with `-g`. When picking with `--skill`, include `validate-finding` alongside `audit-rcp`, `code-review-rcp`, or `docs-review-rcp`.

## License

[MIT](LICENSE)
