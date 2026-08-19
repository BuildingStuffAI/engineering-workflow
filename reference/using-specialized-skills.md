# Use specialized skills and plugins instead of reinventing them

## Contents
- Check before you hand-roll
- What's usually already available
- Closing the loop

## Check before you hand-roll

**Before improvising a process inline for a well-known, well-solved problem
(debugging methodology, plan-then-execute workflows, code review, TDD,
UI/design work), search the skill listing/tool search for a name matching
the problem domain — this is a concrete action, not a mental check.** Don't
rely on remembering what's installed; a matching skill you don't invoke is
the same as it not existing. A maintained, battle-tested skill beats an
ad-hoc version of the same thing written from scratch mid-task. This applies
recursively: this skill itself is exactly that kind of reusable, installable
unit — treat other teams'/authors' equivalents the same way.

**REQUIRED SUB-SKILL:** Use the installed test-driven-development skill (or
equivalent) whenever writing a new function/feature, instead of following
only this skill's summary in [verification.md](verification.md) — the
dedicated skill enforces the write-test-first order; this file's version is
the fallback for when nothing else is installed, not the preferred path.

## What's usually already available

- **Planning**: Claude Code's own Plan mode/agent for structured
  implementation plans before code changes.
- **Ambiguous/creative asks**: a brainstorming skill (e.g.
  `superpowers:brainstorming`) for exploring intent and requirements before
  committing to a plan — pairs with "Clarify before you plan" in
  [planning-and-scope.md](planning-and-scope.md).
- **Code review**: bundled `/code-review` and `/security-review` skills for
  reviewing a diff or pending changes. Run `/security-review` specifically
  whenever a change touches auth, secrets, user input handling, payments, or
  anything crossing a trust boundary — don't fold that into the general
  hard-TL pass and hope it's covered, and don't skip it because the diff
  looks small or the fix seems obvious: doing the equivalent scrutiny
  yourself instead of running the tool is exactly the self-judged-size skip
  the hard-TL gate already forbids, applied to a different gate. Catching
  the actual bug by hand doesn't retroactively make skipping the tool the
  right call — say so out loud if you think this one genuinely doesn't need
  it, rather than deciding quietly.
- **Code comprehension**: a skill that gates on actually understanding
  AI-generated code (algorithms, backend/DB, frontend) before accepting it,
  where available — catches "looks good, ship it" acceptance of code nobody
  actually verified.
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
- **Multi-agent orchestration**: a workflow/orchestration tool capable of
  running several independent subagents (e.g. diverse review lenses) and
  synthesizing their results, where available — turns a single ad-hoc
  review subagent into a real panel instead of one perspective.
- **UI/design work**: a frontend-design skill for distinctive visual design
  decisions, and a dataviz skill for charts/dashboards, where available.

Don't assume any of these are installed — check what's actually available
(skill listing, `/plugin` menu) before relying on one, and say so if something
the task needs isn't available rather than silently working around its
absence.

## Prefer delegation for the hard-TL review

The hard-TL adversarial review before a PR ([git-and-safety.md](git-and-safety.md))
is the clearest case where this skill should stand on other tools' shoulders
rather than own the whole technique itself. In order of preference, use
whatever of these actually exist in the current environment:

1. **A dedicated code-review skill/plugin**, run at its highest effort
   level, for the mechanical pass (bugs, simplification, efficiency).
2. **A multi-agent orchestration tool**, to run 2-3 *diverse-lens* reviewers
   in parallel (e.g. correctness/"what else reaches this state",
   security/data-integrity, would-existing-tests-catch-a-regression) plus a
   synthesis step that merges MUST-FIXes — see the "diverse-lens" wording
   in git-and-safety.md for why more identical clones isn't better, but a
   few distinct lenses is.
3. **A code-comprehension-verification skill**, additionally, whenever the
   diff contains non-trivial AI-generated logic (an algorithm, a query, a
   migration, a non-trivial component) — separate from "is this correct" is
   "does anyone actually understand why," and a skill built for that catches
   a different failure mode than a review does.
4. **The inline single-subagent instructions in git-and-safety.md**, as the
   fallback when none of the above are available — that text stays in this
   skill precisely so it still works standalone, with nothing else
   installed, not as the preferred path when something better exists.

This is the same principle as the rest of this file, made concrete for the
one step in this skill most likely to be reinvented worse than tools that
already exist for it.

## Closing the loop

- Review your own diff for regressions and security issues (injection, XSS,
  and similar) before reporting done — fix anything you find immediately.
- Update docs/memory when a new rule or pattern was needed this session, so
  the next session works better, not just faster.
- Warn proactively and write a compact handoff note before a context limit
  hits mid-task.
- Get it right on a branch/scratch copy first, then promote — don't iterate
  against production or a shared default branch directly.
