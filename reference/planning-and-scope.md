# Planning and scope

## Contents
- Clarify before you plan
- Plan before code
- Architecture decisions, explained plainly
- Break missions into small, verified steps
- Read the reference completely before implementing
- Keep the user in the loop with small increments
- Modularity and one-action functions
- File-creation approval

## Clarify before you plan

An underspecified request planned confidently is still underspecified — it
just looks finished. Before planning, name what's actually ambiguous (which
of 2+ real behaviors was meant, an unstated scale/scope, a missing edge
case) and ask, rather than picking the most-likely reading and running with
it. This matters most for whoever wrote the request with the least
engineering background — they're the least able to catch a wrong guess by
reading a diff. A guess that turns out wrong costs more (redone work, a
regressed decision) than one clarifying question up front.

Don't over-apply this: a request with one obvious reading doesn't need a
question invented for it. The test is "would two competent engineers
reasonably disagree about what was meant," not "is there any conceivable
alternative."

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

## Architecture decisions, explained plainly

When a step has a real design choice (which library, which data model, sync
vs. async, build-vs-reuse, an approach with a scaling/cost/security
tradeoff) — not a mechanical one with a single obvious answer — name at
least one alternative, state the tradeoff in plain language (cost, speed,
complexity to maintain, risk), and give a recommendation before writing
code. "Plain language" means a reader with no engineering background could
follow *why*, not just *what*. This is a small tax on a genuine fork in the
road, not a ceremony for every line — most steps have one obvious way to do
them and don't need this.

Treat this as the mechanism that lets someone who doesn't know the
domain still make the call, or delegate it back knowingly: state the
recommendation, don't just silently pick one and move on, unless the
choice is truly inconsequential (either way is fine, revisit later costs
nothing).

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

## Red flags — modularity

| Excuse | Reality |
|---|---|
| "I'll just add a branch/param to this existing function" | A new case is a new action — new function, unless it's genuinely the same operation. |
| "It's only N lines, splitting it out would be over-engineering for this" | Size isn't the test — a 2-line branch that adds a new case is still a new action. If the user explicitly asked for this exact shape, say so out loud and name the tradeoff (see "Architecture decisions, explained plainly") rather than silently complying or silently overriding. |
| "It's easier to keep it in one file for now" | A distinct concern gets its own file even if "for now" feels faster. |
| "While I'm in here, I'll also improve/clean up X" | Not requested — every changed line should trace to the actual ask. |
| "I'll refactor/modularize this at the end" | Do it in the same edit — a deferred cleanup is a cleanup that doesn't happen. |

## File-creation approval

A new file is a structural decision. If it was already named in an approved
plan, it's approved — don't re-ask per file. Otherwise ask first, especially
for files the user hasn't seen proposed yet.
