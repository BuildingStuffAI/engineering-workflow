# Git, destructive actions, and secrets

## Contents
- Destructive actions
- Secrets and review before push
- Commit/push freely, gate PRs explicitly
- Hard-TL adversarial review before opening a PR
- Per-repo conventions differ

## Destructive actions

- Before any destructive git op (`reset --hard`, `checkout .`/`restore .`,
  `clean -f`, force-push, branch delete), run `git status` first, and stash or
  commit anything at risk rather than discarding it.
- Never force-push to a shared/default branch. Never skip hooks
  (`--no-verify`) or bypass signing without an explicit ask.
- Unfamiliar state (a stray branch, an uncommitted file, a lock file) is
  probably someone's in-progress work — investigate before deleting or
  overwriting it, don't assume it's junk.
- **If a destructive action turns out to already be underway or to have
  already happened unexpectedly** (wrong scope, wrong target, a bug that
  bypassed an explicit stop instruction) — the first move is containment:
  stop, report exactly what happened and its actual blast radius, and wait
  for direction. Don't keep executing further steps or gathering more
  evidence while the risk is still live — "let me confirm what happened
  first" is not a reason to take another action before stopping.

## Secrets and review before push

Before committing or pushing, check what's actually staged — don't blanket
`git add -A`/`git add .` without reviewing the result. If anything looks like
a credential or secret, even in a file with an innocuous name, open it and
confirm before it goes anywhere.

## Commit/push freely, gate PRs explicitly

- Commit and push to a branch per logical change, not batched at the end.
- **Never open a PR, merge, or take any other externally-visible action just
  because a repo's documented delivery path implies it's next.** Committing
  and pushing a branch is fine by default; opening the PR (or pushing to
  `main` where that's convention) is a separate, explicit ask every time —
  even mid-session, even if an earlier PR this session was authorized. A
  green light once is not a standing green light.

## Hard-TL adversarial review before opening a PR

**A same-context self-review is not the hard-TL review, no matter what you
call it.** Re-reading your own diff yourself, in the same context you wrote
it in, and labeling that pass "a hard-TL review" or "an adversarial review"
does not satisfy this gate — it's the exact self-confirming review this gate
exists to replace, wearing the gate's name. The defining requirement is a
**separate dispatch**: a distinct subagent/tool invocation that starts with
no context from your own work and is only given the diff, the root-cause
story, and the checklist below. If you didn't actually dispatch something
separate, you haven't done this yet, regardless of how the review you did
run is described.

Before opening a PR (once tests/tsc/build are green and the diff is what you
intend to ship), run one adversarial review pass with no context from your
own work, told explicitly to act as a very hard, skeptical tech lead trying
to find real problems, not rubber-stamp. **Check
[using-specialized-skills.md](using-specialized-skills.md)'s "Prefer
delegation for the hard-TL review" first** — if a code-review skill,
multi-agent orchestration tool, or code-comprehension skill is available in
this environment, use those instead of hand-rolling the pass described
below. What follows is the fallback shape for when none of those exist, and
the concrete checklist any of those tools should still be pointed at:

Give it the diff (or `git show`/`git diff <base>`), the root-cause story, and
a concrete checklist of specific things to hunt for (not "review this" —
name the actual risk vectors: wrong-value substitutions, missed sibling
instances of the same bug elsewhere in the repo, whether existing tests would
actually catch a regression of this exact fix, stale-base/merge risk, and
anything scope-adjacent that changed but shouldn't have). Require it to
verify claims itself (grep, re-run commands) rather than trust the diff's own
commit message, and to report findings as MUST-FIX vs. nice-to-have vs.
non-issues-checked-and-ruled-out — not a bare "looks good."

If using a multi-agent orchestration tool, prefer 2-3 reviewers with
*distinct* lenses (e.g. correctness/"what else reaches this state",
security/data-integrity, would-existing-tests-catch-a-regression) over more
copies of the same prompt — identical clones converge on the same findings
and mostly just multiply cost; diversity catches failure modes redundancy
can't. When running 2+ passes, also vary the order/framing each one sees
(e.g. which file it reads first) even across the same lens — reading file A
before B forms hypotheses that filter what gets noticed in B, and reversing
the order surfaces different findings from an otherwise-identical reviewer.
Weight a finding multiple passes converge on independently over a singleton
flag from just one; don't merge every pass's list into one undifferentiated
MUST-FIX pile treating all findings as equally confident.

- Why: a single self-review after writing the fix tends to confirm the
  author's own framing rather than challenge it. An independent, adversarial
  pass with no prior context catches the class of bug a same-context review
  misses — e.g. a plausible-looking token/value substitution that's subtly
  wrong, or a second instance of the same root cause elsewhere in the repo.
  This is cheap (one subagent call) relative to a bug re-surfacing in review
  or production. See `sherlock-hard-tl-review-pattern` project memory for
  concrete prior catches from this pattern (a regex fix, a fallback chain, a
  cross-repo redirect fix, a billing double-charge, a discarded-async-result
  bug) — the recurring blind spot it catches is "what/who else reaches this
  state."
- How to apply: **the default is to always run it**, once per PR/push, every
  time — including a small follow-up push on a branch already reviewed
  earlier in the session. Self-judging a diff as "too small to need it" is
  the exact silent-skip failure mode this rule exists to close; if a diff
  genuinely seems trivial enough to skip (a pure rename in a single call
  site, a comment-only change), say so out loud to the user before opening
  the PR and get their agreement — don't decide it quietly and skip. Fix
  whatever the review confirms as real, re-verify, then proceed to the
  PR/push gate above.

## Per-repo conventions differ

Sibling repos in the same family/org can have genuinely different delivery
conventions — one may require branch → PR → review, another may push straight
to `main`. Confirm per-repo, don't generalize one repo's stated rule to its
siblings without checking.

## Shared skill/plugin repos are shared working directories too

A skill or plugin's own repo (e.g. this one) gets edited directly — no
worktree isolation — often by more than one concurrent Claude session at
once, since nothing about "editing a skill" signals "treat this like a
shared codebase" the way a project repo does. It is one. Before committing
here: `git status` to see what's already sitting uncommitted (another
session's in-progress edit, not junk — see "Destructive actions" above), and
`git fetch`/`git pull` before pushing, since another session may have pushed
its own commit while you were working. A push rejected for "fetch first" is
expected here, not exceptional — merge properly (never blindly take "ours"
or "theirs" on a file two sessions both touched, see
[merge-conflicts.md](merge-conflicts.md)) rather than force-pushing over it.

## Merge conflicts

A conflict at merge/rebase time is not a fast keystroke problem — it's a real
code change with two intents colliding, and it gets the same plan-first,
verify-after treatment as everything else in this skill. See
[merge-conflicts.md](merge-conflicts.md) for the full procedure: detect
before assuming clean, never blindly take "ours"/"theirs" on a whole file,
re-verify and re-review after resolving, escalate when the other side's
intent is unclear.
