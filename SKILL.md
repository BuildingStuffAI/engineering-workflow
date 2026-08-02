---
name: engineering-workflow
description: General software-engineering workflow discipline for any repo — plan before code, one-action functions, test before/during/after, git/PR gating, destructive-action safety, secret hygiene, scope discipline, version-bump hygiene, file-creation approval, ticket discipline. Use at the start of and throughout any non-trivial engineering task (fix, feature, refactor) in any language or codebase.
---

# Engineering Workflow

Default operating mode for engineering work, distilled from repeated real
corrections, not theory. A repo's own `CLAUDE.md`/`docs/` wins over this when
they conflict — this is the floor, not a ceiling.

**Tradeoff:** biases toward caution over speed. Use judgment on trivial work —
don't ceremony-gate a typo fix.

## 1. Plan before code

Never edit before planning: what's affected, what's the blast radius, what's
the risk. Before touching or removing a function, find every caller and every
connected file — "I changed the function" isn't done until you know what else
touches it. Right-size the plan to the task.

Break a multi-part mission into small, independently-verifiable steps.
Finish and verify one step before starting the next, even when nothing
technically blocks doing them in parallel — don't duplicate an unverified
change across multiple targets (repos, files, environments) and find out
together whether it worked. Slow and confirmed beats fast and unconfirmed.

## 2. Stay modular, stay scoped

- New feature/helper/"one more action" → new function, new file if it's a
  distinct concern. Don't grow existing functions to cover unrelated cases.
- One function = one action, always.
- Prefer a few similar lines over a premature abstraction.
- Don't add error handling, validation, abstractions, or config knobs for
  scenarios nobody asked for. Every changed line should trace to the actual
  request — don't "improve" adjacent code while you're in there.
- Match existing conventions even where you'd choose differently. No dead code,
  no commented-out code, no backwards-compat shims for internal-only code.

## 3. Don't create files without approval

A new file is a structural decision. If it was already named in an approved
plan, it's approved — don't re-ask per file. Otherwise ask first, especially
for files the user hasn't seen proposed yet.

## 4. Test before, during, and after

- **Before**: confirm the current baseline is actually green.
- **During**: new function/feature gets its test alongside it — real edge
  cases, not one happy path.
- **After**: re-run the full suite, not just the new test. For UI/behavior
  changes, verify in the running app if at all possible; if you can't, say so
  rather than implying you did. "Verified individually" ≠ "verified together" —
  do one combined regression pass at the end.

## 5. Destructive actions and git safety

- Before any destructive git op (`reset --hard`, `checkout .`/`restore .`,
  `clean -f`, force-push, branch delete), run `git status` first, and stash or
  commit anything at risk rather than discarding it.
- Never force-push to a shared/default branch. Never skip hooks (`--no-verify`)
  or bypass signing without an explicit ask.
- Unfamiliar state (a stray branch, an uncommitted file, a lock file) is
  probably someone's in-progress work — investigate before deleting or
  overwriting it, don't assume it's junk.

## 6. Secrets and review before push

Before committing or pushing, check what's actually staged — don't blanket
`git add -A`/`git add .` without reviewing the result. If anything looks like
a credential or secret, even in a file with an innocuous name, open it and
confirm before it goes anywhere.

## 7. Git: commit/push freely, gate PRs explicitly

- Commit and push to a branch per logical change, not batched at the end.
- **Never open a PR, merge, or take any other externally-visible action just
  because a repo's documented delivery path implies it's next.** Committing
  and pushing a branch is fine by default; opening the PR (or pushing to
  `main` where that's convention) is a separate, explicit ask every time —
  even mid-session, even if an earlier PR this session was authorized.
- Sibling repos can have different delivery conventions. Confirm per-repo,
  don't generalize one repo's rule to another.

## 8. Version and release hygiene

- Bump the version marker (versionCode/semver/build number) whenever a change
  affects shipped behavior, even a no-code-change redelivery — the point is
  recognizability between drops in analytics/crash tooling.
- Bumping a version number is a text edit, not a build. Don't build or upload
  an artifact unless explicitly asked, and don't narrate a version bump as if
  it shipped something.
- Verify release-readiness with an actual build/test run, not a static config
  check.

## 9. Ticket discipline

Create or update a tracking ticket for real changes, scoped to everything the
change actually touches — a change that ripples into a connected component
belongs in one ticket, not fragmented across several undersold ones.

## 10. Load what the task needs

Decide which tools/skills/docs the task needs and load only that. If
something needed is missing, flag it — don't silently work around it.

## 11. Close the loop

- Review your own diff for regressions and security issues (injection, XSS,
  and similar) before reporting done — fix anything you find immediately.
- Update docs/memory when a new rule or pattern was needed, so the next
  session works better, not just faster.
- Warn proactively and write a compact handoff note before a context limit
  hits mid-task.
- Get it right on a branch/scratch copy first, then promote — don't iterate
  against production or a shared default branch directly.

---

**Working if:** plans precede diffs, diffs stay scoped to the ask, nothing
destructive happens without a status check, nothing ships or merges without an
explicit go-ahead, and regressions get caught before the user does.
