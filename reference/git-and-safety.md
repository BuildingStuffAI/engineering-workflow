# Git, destructive actions, and secrets

## Contents
- Destructive actions
- Secrets and review before push
- Commit/push freely, gate PRs explicitly
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

## Per-repo conventions differ

Sibling repos in the same family/org can have genuinely different delivery
conventions — one may require branch → PR → review, another may push straight
to `main`. Confirm per-repo, don't generalize one repo's stated rule to its
siblings without checking.
