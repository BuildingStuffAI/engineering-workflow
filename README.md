# engineering-workflow

A Claude Code skill/plugin encoding general software-engineering workflow
discipline — plan before code, mission decomposition into small verified
steps, one-action functions, evidence-before-claims verification, bounded
loop-until-verified, destructive-action safety, secret hygiene, git/PR
gating, merge-conflict resolution discipline, version-bump hygiene,
file-creation approval, ticket discipline, and using specialized
skills/plugins (debugging, TDD, planning, code review, design) instead of
reinventing them. App-agnostic: no project-specific content, usable in any
repo/language.

`SKILL.md` is a compact index; the detail lives one level deep in
[`reference/`](reference/):

- [`planning-and-scope.md`](reference/planning-and-scope.md)
- [`verification.md`](reference/verification.md)
- [`git-and-safety.md`](reference/git-and-safety.md)
- [`merge-conflicts.md`](reference/merge-conflicts.md)
- [`release-and-tickets.md`](reference/release-and-tickets.md)
- [`using-specialized-skills.md`](reference/using-specialized-skills.md)
- [`qa-and-production-verification.md`](reference/qa-and-production-verification.md)
- [`self-improvement.md`](reference/self-improvement.md) — how this skill
  compounds: per-area findings logs distilled periodically into this
  skill's own rules, plus a periodic outward-looking frontier scan for new
  AI-assisted engineering techniques/tools

### Findings logs (`reference/findings/`)

Per-area, low-friction logs — created on demand, not pre-provisioned. See
[`self-improvement.md`](reference/self-improvement.md) for the format and
the distillation process that keeps these from growing forever.

- [`findings/qa.md`](reference/findings/qa.md) — seeded, active
- [`findings/merge-conflicts.md`](reference/findings/merge-conflicts.md) — seeded, active

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
