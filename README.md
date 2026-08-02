# engineering-workflow

A Claude Code skill/plugin encoding general software-engineering workflow
discipline — plan before code, one-action functions, test before/during/after,
destructive-action safety, secret hygiene, git/PR gating, version-bump hygiene,
file-creation approval, ticket discipline. App-agnostic: no project-specific
content, usable in any repo/language.

See [`SKILL.md`](SKILL.md) for the full rule set.

## Install (team)

```
/plugin marketplace add BuildingStuffAI/engineering-workflow
/plugin install engineering-workflow@engineering-workflow-marketplace
```

## Install (personal, no plugin system)

Copy `SKILL.md` to `~/.claude/skills/engineering-workflow/SKILL.md` — it works
as a plain skill with no plugin machinery required.

## Updating

After pulling changes, bump `version` in `.claude-plugin/plugin.json` and push.
Installed users get the update via `/plugin marketplace update` +
`/plugin update engineering-workflow@engineering-workflow-marketplace`.
