# Use specialized skills and plugins instead of reinventing them

## Contents
- Check before you hand-roll
- What's usually already available
- Closing the loop

## Check before you hand-roll

Before improvising a process inline for something that's a well-known,
well-solved problem (debugging methodology, plan-then-execute workflows, code
review, TDD, UI/design work), check whether a specialized skill or plugin
already covers it — built-in, previously installed, or available from a
marketplace. A maintained, battle-tested skill beats an ad-hoc version of the
same thing written from scratch mid-task. This applies recursively: this
skill itself is exactly that kind of reusable, installable unit — treat other
teams'/authors' equivalents the same way.

## What's usually already available

- **Planning**: Claude Code's own Plan mode/agent for structured
  implementation plans before code changes.
- **Code review**: bundled `/code-review` and `/security-review` skills for
  reviewing a diff or pending changes.
- **Debugging**: a systematic root-cause-first methodology (e.g. the
  `systematic-debugging` skill in the
  [Superpowers marketplace](https://github.com/obra/superpowers-marketplace))
  — root cause before fixes, not symptom patching.
- **TDD / plan-execute / git worktree parallelism / structured code-review
  handoff**: the wider Superpowers skill set (`test-driven-development`,
  `writing-plans`, `executing-plans`, `requesting-code-review`,
  `receiving-code-review`, `using-git-worktrees`, `dispatching-parallel-agents`,
  `subagent-driven-development`) — install via
  `/plugin marketplace add obra/superpowers-marketplace` then
  `/plugin install superpowers@superpowers-marketplace` if not already present.
- **UI/design work**: a frontend-design skill for distinctive visual design
  decisions, and a dataviz skill for charts/dashboards, where available.

Don't assume any of these are installed — check what's actually available
(skill listing, `/plugin` menu) before relying on one, and say so if something
the task needs isn't available rather than silently working around its
absence.

## Closing the loop

- Review your own diff for regressions and security issues (injection, XSS,
  and similar) before reporting done — fix anything you find immediately.
- Update docs/memory when a new rule or pattern was needed this session, so
  the next session works better, not just faster.
- Warn proactively and write a compact handoff note before a context limit
  hits mid-task.
- Get it right on a branch/scratch copy first, then promote — don't iterate
  against production or a shared default branch directly.
