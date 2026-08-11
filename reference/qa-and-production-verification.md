# QA and production verification

Use this after code has already merged/shipped, to answer "does this actually
work, right now, for real users" — a different question from "did the PR's
own tests pass," which the [verification](verification.md) area already
covers pre-merge. Trigger this area when: several PRs merged close together
and you need to confirm none of them collided; a deploy pipeline's success
is ambiguous (see "Deploy confirmation" below); or someone asks to "verify
this works in production" / "QA this ticket."

## The three things code review alone cannot tell you

1. **Whether it actually deployed.** A merged PR and a live deploy are not
   the same fact — confirm both (see "Deploy confirmation" below).
2. **Whether concurrent changes collided.** Tests passing on each PR's own
   branch says nothing about what happens when several land within minutes
   of each other and touch the same file (shared CSS, a shared hook). Diff
   the *real* merge result, not each branch against its own base.
3. **Whether it works in the actual running app.** A correct diff and a
   green test suite can still fail against real data shapes, real auth
   state, or real timing — a live check catches what a mock can't.

## Process

1. **Confirm the ticket → PR → merge → deploy chain**, in that order, before
   touching code-level analysis. Use `gh pr view` / `gh pr list` to find the
   actual merged PR per ticket (a ticket number is not a PR number). Then
   confirm the deploy:
   - **Deploy confirmation, don't assume merge = deployed.** If deploys
     trigger on push-to-`main` with a `concurrency` group and
     `cancel-in-progress: true` (a common pattern to avoid overlapping
     deploys), a burst of rapid merges can cancel every deploy run except
     the last. Check `gh run list --workflow=<deploy>.yml` for
     `cancelled` conclusions among your recent merges. A cancelled run is
     **not** a failed deploy of that change by itself — if a *later* run in
     the same burst succeeded, it deployed from `main`'s tip at that later
     point, which includes every commit merged before it. Confirm this with
     `git merge-base --is-ancestor <your-commit> <deployed-sha>` (get
     `<deployed-sha>` from the successful run's `headSha`), not by assuming.
     Cross-check with the live asset's `Last-Modified` header (or
     equivalent) against the deploy's completion timestamp as a second
     confirmation.
   - Flag this pattern to the team if found: it means "merged" and "these 4
     changes are live" can silently diverge whenever the *last* merge in a
     rapid burst happens to fail its build — nothing surfaces that failure
     against the earlier, cancelled merges' PRs. Worth a process fix
     (serialize deploys, or alert on any cancelled run in a burst) if it
     recurs.

2. **For each ticket/PR, re-verify against the CURRENT merged tip, not the
   PR's own branch.** Read the real ticket requirement again (don't trust a
   summary of it) and check the shipped code still satisfies it — code
   review skills apply here (see
   [using-specialized-skills.md](using-specialized-skills.md)). If several
   PRs touched the same file, specifically hunt for cascade/ordering
   collisions (CSS specificity + source order, shared hook state, shared
   color tokens) — diffing each branch against its own base won't show
   this; diff the real merge commit, or read the final file directly via
   `git show <tip-sha>:<path>`.

3. **Run the actual regression tests against the merged tip**, not just
   trust that they passed in CI on each branch — a clean merge can still
   combine two individually-correct branches into a broken result if a test
   only exists on one side. Prefer running the suite in a disposable
   worktree checked out at the tip, so you don't disturb a shared working
   directory another session might be using.

4. **Do a live check in the running app for anything a static diff can't
   settle** — an actual click-through against production (or the closest
   available real environment), not a mock. Use an adversarial mindset per
   ticket: don't just confirm the happy path the ticket named, try the
   states around it (a profile with no data for this field, a second
   click, a page reload). If a live viewport/environment limitation blocks
   part of the check (e.g. a sandboxed browser that can't resize), say so
   explicitly rather than skip silently — note exactly what wasn't checked
   and why, so a human knows the gap exists.
   - **If the thing you need to click only renders conditionally** (a
     section that appears only when certain data exists, an empty vs.
     populated state), don't manually sample real records hoping to find
     one — after 1-2 misses, search for it directly (e.g. an
     accessibility-tree/DOM search tool, or a direct query against whatever
     the app's data source is) rather than continuing to guess. Manual
     sampling wastes turns and can wrongly end in "couldn't find one to
     test" when a real instance existed all along.
   - **If the environment has real usage limits** (credits, quotas, a real
     paid account), prefer checking against data that already exists
     (search history, prior results) before triggering something that
     consumes a new unit — a live check should confirm behavior, not run up
     costs it doesn't need to.

5. **Log every finding — pass or reject — before moving on.** See
   [qa-findings-log.md](qa-findings-log.md): a real fix suggestion or root
   cause, once found, should never be lost to a single conversation. Append
   in the format that file describes, whether the verdict was PASS or
   REJECT — a documented PASS with its evidence is what lets the next QA
   pass skip re-deriving the same thing from scratch, and prevents "already
   verified, still worth re-checking against live prod before closing" from
   turning into blind re-litigation each time (see the "known non-issue"
   pattern in the log).

## Name your evidence tier — never report a bare "PASS"

Code-read, executed-test-on-the-merged-tip, and live-UI-click-through are
three different strengths of evidence, and a verdict is incomplete without
saying which one it rests on. State it explicitly per item, e.g.:

- "PASS (code + executed test only)" — you read the merged code and/or ran
  its test suite against the real merged tip, but did not click it live.
- "PASS (live-confirmed)" — you actually drove the real running app and
  observed the behavior yourself.

This matters because the first two steps of this process (deploy
confirmation, cross-PR collision check) and step 3 (running tests) can all
come back clean while step 4 (the actual live check) never happened for a
given item — a plain "PASS" doesn't tell a reader which of those happened.
Someone relying on the verdict needs to know whether "verified" means
"the code and its own tests look right" or "I watched it work" — those
justify different amounts of confidence, and only one of them is what
"verify this works in production" is actually asking for by default. If
asked to verify production behavior and you only have code/test-level
evidence for some items, say so and go get the live evidence rather than
letting a tier-1 verdict pass as if it were tier-3.

## Suggested agent split (when using subagents for this)

A single QA pass benefits from separating concerns rather than one agent
doing everything:

- **One agent per ticket/PR** for the static/code-level re-verification
  (step 2-3 above) — these can run in parallel/background since they're
  read-only and independent.
- **You, directly, for the live UI check** (step 4) — browser automation
  tools are often stateful (one shared tab/session) and contend if run from
  multiple parallel agents at once; do this part yourself or serially.
- **A synthesis pass** (you, or one final agent) that reads all the
  per-ticket verdicts, drafts the findings-log entry, and decides what (if
  anything) needs a follow-up ticket.

Don't reach for heavier multi-stage orchestration than the task needs — a
handful of parallel one-shot verification agents plus your own live check is
usually enough; save deeper orchestration for when the number of
tickets/PRs is large enough that manual tracking becomes the bottleneck.

## Red flags in your own QA output

- Reporting "PASS" because a PR's own tests passed on its own branch,
  without checking the actual merged/deployed result.
- Treating "merged" as proof of "deployed" without checking the deploy
  workflow run itself.
- Skipping the live check because the code review "looked right" — code
  review and live behavior are different kinds of evidence, per
  [verification.md](verification.md)'s evidence-before-claims gate.
- Silently dropping a finding that seems minor "since it's not blocking" —
  log it anyway (see the findings log's non-blocking-gap entries) so it
  isn't rediscovered from zero next time.
- Reporting a bare "PASS" across several items without naming which
  evidence tier backs each one — see "Name your evidence tier" above. If a
  human has to ask "did you actually click it, or just check the code?"
  after your verdict, the verdict was underspecified.
- Giving up on a live check for a conditionally-rendered element after
  one or two samples come up empty, instead of searching for a real
  instance directly.
