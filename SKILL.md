---
name: engineering-workflow
description: General software-engineering workflow discipline for any repo — plan before code, mission decomposition into small verified steps, one-action functions, evidence-before-claims verification, bounded loop-until-verified, destructive-action safety, secret hygiene, git/PR gating, merge-conflict resolution discipline, version-bump hygiene, file-creation approval, ticket discipline, and using specialized skills/plugins (debugging, TDD, planning, code review, design) instead of reinventing them. Use at the start of and throughout any non-trivial engineering task (fix, feature, refactor) in any language or codebase, including when resolving a merge conflict.
---

# Engineering Workflow

Default operating mode for engineering work, distilled from repeated real
corrections, not theory. A repo's own `CLAUDE.md`/`docs/` wins over this when
they conflict — this is the floor, not a ceiling.

**Tradeoff:** biases toward caution over speed. Use judgment on trivial work —
don't ceremony-gate a typo fix.

## The two hard gates

**No code before a plan.** What's affected, blast radius, risk — right-sized
to the task. See [reference/planning-and-scope.md](reference/planning-and-scope.md).

**No completion claim without fresh verification evidence run in this
exchange.** "Should work" is not evidence. See
[reference/verification.md](reference/verification.md).

Everything below expands on these two, plus adjacent discipline.

## Areas

- **[Planning and scope](reference/planning-and-scope.md)** — plan before
  code; break missions into small, independently-verified steps; read a
  reference/spec completely before implementing; keep the user in the loop
  with small increments; one function = one action; modularity and scope
  discipline (no unrequested abstractions); file-creation needs approval.
- **[Verification](reference/verification.md)** — the evidence-before-claims
  gate; test before/during/after; loop-until-verified, but bounded (real
  fresh check each time, informed retries, hard attempt cap); red flags to
  catch in your own output.
- **[Git and safety](reference/git-and-safety.md)** — destructive-action
  safety (`git status` before anything destructive, never force-push to
  shared branches); secrets/review before push; commit and push to a branch
  freely, but gate PRs and pushes-to-main as a separate explicit ask every
  time; per-repo delivery conventions differ, confirm don't assume.
- **[Merge conflicts](reference/merge-conflicts.md)** — treat a conflict as a
  task, not a keystroke: plan before resolving, never blindly take "ours" or
  "theirs" on a whole file, re-verify and re-review after resolving, escalate
  when the other side's intent is unclear.
- **[Release and tickets](reference/release-and-tickets.md)** — bump version
  markers for shipped-behavior changes; a version bump is a text edit, not a
  build — don't build/ship unless asked; ticket discipline scoped to the
  whole connected change.
- **[Using specialized skills](reference/using-specialized-skills.md)** —
  check for and use an existing debugging/TDD/planning/code-review/design
  skill or plugin before hand-rolling the equivalent inline; close the loop
  (self-review, update docs, proactive context-limit handoff, get it right on
  a branch before promoting to production).

---

**Working if:** plans precede diffs, diffs stay scoped to the ask, nothing
destructive happens without a status check, nothing ships or merges without
an explicit go-ahead, and every completion claim has fresh evidence behind it.
