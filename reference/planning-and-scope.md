# Planning and scope

## Contents
- Plan before code
- Break missions into small, verified steps
- Read the reference completely before implementing
- Keep the user in the loop with small increments
- Modularity and one-action functions
- File-creation approval

## Plan before code

Never edit before planning: what's affected, what's the blast radius, what's
the risk. Before touching or removing a function, find every caller and every
connected file — "I changed the function" isn't done until you know what else
touches it. Right-size the plan to the task: a one-line plan for a typo, a
full plan for a multi-file change. Don't ceremony-gate trivial work.

When the plan has acceptance criteria or a spec to satisfy, write each
criterion as something a command can check ("returns matching results
within 200ms," "field is non-null after Z") rather than prose to be
re-read at the end. Decide during planning what will actually be run to
prove each criterion true — this is what the evidence gate in
[verification.md](verification.md) gets run against later; leaving it
implicit invites eyeballing the spec at completion time instead of
checking it against something real.

## Break missions into small, verified steps

Break a multi-part mission into small, independently-verifiable steps. Finish
and verify one step before starting the next, even when nothing technically
blocks doing them in parallel. Don't duplicate an unverified change across
multiple targets (repos, files, environments) and find out together whether
it worked — verify on the first target, then replicate the proven change.
Slow and confirmed beats fast and unconfirmed.

## Read the reference completely before implementing

When asked to match an existing pattern, a design spec, a reference
implementation, or another file's exact behavior: read the whole thing before
writing code, not a skim. Don't assume "that part probably doesn't matter" —
list every difference between the reference and your draft, however small,
before calling it done. This applies to a design mockup, an API you're
integrating against, or a sibling implementation you're porting a fix from.

## Keep the user in the loop with small increments

Don't produce one huge diff sprung at the end of a long silent session. Land
and surface changes in small, reviewable increments so the user can actually
review what changed, not just trust a final summary. If a mission is large
enough that this skill's own rules feel like overhead, that's a signal the
mission should be split into smaller missions, not that the rules should be
skipped.

## Modularity and one-action functions

- New feature/helper/"one more action" → new function, new file if it's a
  distinct concern. Don't grow existing functions to cover unrelated cases.
- One function = one action, always.
- Prefer a few similar lines over a premature abstraction.
- Don't add error handling, validation, abstractions, or config knobs for
  scenarios nobody asked for. Every changed line should trace to the actual
  request — don't "improve" adjacent code while you're in there.
- Match existing conventions even where you'd choose differently. No dead
  code, no commented-out code, no backwards-compat shims for internal-only
  code.

## File-creation approval

A new file is a structural decision. If it was already named in an approved
plan, it's approved — don't re-ask per file. Otherwise ask first, especially
for files the user hasn't seen proposed yet.
