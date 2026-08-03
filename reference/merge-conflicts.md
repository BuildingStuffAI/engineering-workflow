# Merge conflicts: treat resolution as a task, not a keystroke

## Contents
- Detect before you assume clean
- Resolution is a mission, not a shortcut
- Never blindly pick a side
- Re-verify and re-review after resolving
- Escalate when blast radius is unclear

## Detect before you assume clean

- Before telling the user a PR is ready to merge, check its actual current
  merge state (`gh pr view --json mergeable,mergeStateStatus`), not just "no
  conflicts last I looked." State can change the moment someone else pushes.
- The moment a conflict appears — rebase/merge output, the PR UI, `git status`
  mid-rebase — stop before resolving anything. Treat it like a bug report:
  understand what changed on both sides before touching a single character.

## Resolution is a mission, not a shortcut

A conflict is a real code change with two authors and two intents colliding.
Run it through this skill's own two hard gates, not around them:

- **Plan first.** For each conflicting hunk, read both sides' diffs against
  their common ancestor and understand *why* each side changed — not just
  what the text says. Name the resolution approach (keep both, merge
  semantically, one supersedes the other) before editing, especially when the
  conflict isn't a pure textual overlap (e.g. one side renamed something the
  other side also edited).
- **No completion claim without fresh verification.** Resolving cleanly (no
  conflict markers left) is not the same as resolving *correctly*. It needs
  the same build/test/run pass — see
  [verification.md](verification.md) — as any other change to that code.

## Never blindly pick a side

- Never default to `git checkout --ours`/`--theirs` (or the PR UI's "accept
  current/incoming") on a whole file. That silently discards one side's real
  work. It's only acceptable after you've confirmed the discarded side is
  fully superseded by the side you're keeping — state that confirmation
  explicitly, don't just do it and move on.
- When both sides touch the same function/logic for different reasons (one
  fixes a bug, the other refactors the same area), reconcile both intents in
  the merged result. Don't let a rename silently win and drop the bug fix, or
  vice versa — read both commit messages/diffs to recover each side's intent
  before merging them by hand.

## Re-verify and re-review after resolving

- Rebuild and rerun the relevant checks after resolving. A conflict
  resolution is exactly the kind of edit likely to introduce a subtle
  regression — it's written under time pressure with two versions of the
  truth in front of you at once.
- Run `/code-review` (or the repo's equivalent — see
  [using-specialized-skills.md](using-specialized-skills.md)) on the
  resulting diff before pushing. A conflict resolution is not exempt from
  review just because a tool auto-staged most of the file.
- If the conflict touched something already verified on-device this session
  (UI, animation, behavior), re-verify on-device again. A resolved conflict
  can silently reintroduce the exact bug that was just fixed, if the wrong
  side's hunk ends up winning for that hunk.

## Escalate when blast radius is unclear

- If the conflicting change came from someone else's commit, or you're not
  confident you understand the intent of both sides, stop and ask rather than
  guess — this mirrors the general "unfamiliar state is someone's
  in-progress work" rule in [git-and-safety.md](git-and-safety.md).
- Never force-push a conflict resolution over a shared/default branch's
  history. Resolve on your own branch, verify, then merge/rebase normally —
  the same PR-gating rule that applies to every other push applies here too.
