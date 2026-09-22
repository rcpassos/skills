# skills

Agent skills for Claude Code, Codex, and other [skills](https://skills.sh)-compatible agents. The `-rcp` skills are user-invoked — run them by name (e.g. `/audit-rcp`); the agent won't trigger them on its own.

| Skill | What it does |
| --- | --- |
| [`audit-rcp`](skills/audit-rcp/SKILL.md) | Read-only full-project audit of a local path or GitHub repo: bugs, security, architecture, performance, tech debt. Verified, prioritized findings. |
| [`code-review-rcp`](skills/code-review-rcp/SKILL.md) | Three-axis review (Standards, Spec, Risk) of a PR, branch, or diff, verified and triaged into **Take it** / **Leave it**. |
| [`issue-check-rcp`](skills/issue-check-rcp/SKILL.md) | Validate a GitHub issue before work starts: is it valid, is it useful, what's the risk, how big is it. |
| [`validate-finding`](skills/validate-finding/SKILL.md) | Classify a suspected issue as bug, possible issue, false-positive, or recommendation, backed by evidence. Used by `audit-rcp` and `code-review-rcp` to verify findings; also usable on its own. |

## Install

```bash
npx skills add rcpassos/skills
```

Install one skill with `--skill <name>`; list them first with `--list`; install for your user instead of the project with `-g`. When picking with `--skill`, include `validate-finding` alongside `audit-rcp` or `code-review-rcp`.

## License

[MIT](LICENSE)
