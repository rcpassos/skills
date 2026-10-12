# skills

Agent skills for Claude Code, Codex, and other [skills](https://skills.sh)-compatible agents. The `-rcp` skills are user-invoked — run them by name (e.g. `/audit-rcp`); the agent won't trigger them on its own.

Skills are grouped by category under `skills/<category>/<name>/`.

### General

| Skill | What it does |
| --- | --- |
| [`audit-rcp`](skills/general/audit-rcp/SKILL.md) | Read-only full-project audit of a local path or GitHub repo: bugs, security, architecture, performance, tech debt. Verified, prioritized findings. |
| [`code-review-rcp`](skills/general/code-review-rcp/SKILL.md) | Verified review of a PR, branch, or diff across Standards, Spec, and Risk, with severity, saved decisions, and follow-up checks of fixes. |
| [`issue-check-rcp`](skills/general/issue-check-rcp/SKILL.md) | Validate a GitHub issue before work starts: is it valid, is it useful, what's the risk, how big is it. |
| [`validate-finding`](skills/general/validate-finding/SKILL.md) | Classify a suspected issue as bug, possible issue, false-positive, or recommendation, backed by evidence. Used by `audit-rcp` and `code-review-rcp` to verify findings; also usable on its own. |

### WordPress

| Skill | What it does |
| --- | --- |
| [`wp-plan-rcp`](skills/wordpress/wp-plan-rcp/SKILL.md) | Plan a WordPress site build from Figma via the Figma and Novamira MCPs: clarifying questions, a phased `docs/plan.md`, and `AGENTS.md` plus supporting docs for implementers. |
| [`wp-implement-rcp`](skills/wordpress/wp-implement-rcp/SKILL.md) | Implement one phase of a `wp-plan-rcp` plan on the site, verify it against Figma, and write the handoff for the next session. |

## Install

```bash
npx skills add rcpassos/skills
```

Install one skill with `--skill <name>`; list them first with `--list`; install for your user instead of the project with `-g`. When picking with `--skill`, include `validate-finding` alongside `audit-rcp` or `code-review-rcp`.

## License

[MIT](LICENSE)
