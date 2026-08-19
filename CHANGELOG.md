# Changelog

All notable changes to this skill are recorded here. Versions match
`.claude-plugin/plugin.json`.

## 2.8.0

- Ran 4 pressure-test scenarios against v2.7.0 (fresh subagents, realistic
  time/authority pressure, real file edits). 2 held clean (trivial-fast-path
  stayed lightweight; ambiguous-request correctly asked instead of
  guessing). 2 found real gaps, both fixed:
  - **Modularity gate failed**: a "just add a branch to this existing
    function" request under demo-deadline pressure got waved through —
    traced to the new trivial-fast-path's line-count heuristic being
    gameable. Fast-path test reworded from "a line or two" to "a single
    obviously correct form," and the specific rationalization ("only N
    lines, splitting is over-engineering") added to the modularity
    red-flags table with an explicit say-it-out-loud escape valve.
  - **Security-review gate partially failed**: an open-redirect
    vulnerability was caught and fixed by hand, but `/security-review`
    itself was explicitly skipped as "not worth the overhead" for a small
    diff — same self-judged-size skip the hard-TL gate already forbids, not
    yet forbidden by name for this gate. Fixed in `SKILL.md`'s gates table
    and `using-specialized-skills.md`.
- New findings logs seeded: `findings/planning-and-scope.md`,
  `findings/using-specialized-skills.md`.

## 2.7.0

- Distilled the QA findings log (372 → 75 lines, 4 entries had ballooned to
  60-130 lines each): folded 4 new durable lessons into the reference docs
  — ticket/PR number-coincidence trap, verify-the-actual-artifact before
  calling something a regression (seen twice independently), emulator-vs-
  real-device as distinct evidence tiers, and a test mocking away a bug's
  exact mechanism proving nothing about that bug — then thinned the log to
  pointers, per the skill's own Layer-2 distillation process.
- `self-improvement.md`'s distillation trigger now also fires on a single
  entry exceeding ~30 lines, not just log-wide entry count — the ballooned
  entries above were caught by count (4, under the old 6-8 threshold) but
  were already more expensive to load than 8 tight ones would have been.

## 2.6.0

- Added a trivial-task fast path and made reference-file loading explicitly
  lazy (one file per fired gate, never a batch read at skill-start) —
  addresses reports of heavy upfront context reads before a 1-2 line fix.
- New gates: a new function's test is written the same turn (not "after"),
  a modularity check before squeezing code into an existing function, and
  "search installed skills before hand-rolling" as an explicit action.
- `verification.md` and `planning-and-scope.md` gained rationalization
  tables for the specific excuses that let test-writing and modularity
  slip ("I'll add tests after," "I'll clean this up at the end").
- `using-specialized-skills.md`: TDD skill marked as a REQUIRED SUB-SKILL
  (dispatch it instead of improvising inline); added brainstorming for
  ambiguous/creative asks and an explicit `/security-review` trigger for
  auth/secrets/input/payment-touching changes.
- `planning-and-scope.md`: new "Clarify before you plan" (ask when a
  request has 2+ reasonable readings, rather than guessing) and
  "Architecture decisions, explained plainly" (name the tradeoff and a
  recommendation in plain language for a real design fork, not a
  mechanical choice) sections — aimed at making the skill useful to
  someone without an engineering background, not just faster for one who
  already has it.

## 2.5.0

- Closed a same-context-self-review loophole in the hard-TL gate: a diff
  re-read in the same context it was written in doesn't satisfy the gate
  just because it's labeled "a hard-TL review" — only a genuinely separate
  dispatch does. Found via a paired control/test pressure test (subagents
  with and without the v2.4.0 wording, identical time-pressure scenario).
- First frontier scan, adopted three findings: (1) vary review-pass
  order/framing, not just lens, when running 2+ reviewers, and weight
  multi-pass-convergent findings over singletons; (2) destructive-action
  containment for the during/after moment (already underway or already
  happened), distinct from the existing pre-check; (3) write acceptance
  criteria as agent-checkable commands during planning, not prose re-read
  at completion time.
- Added `CHANGELOG.md` and per-area findings logs for git-and-safety and
  self-improvement, seeded with this release's pressure-test results.

## 2.4.0

- Gates (tests, hard-TL, evidence, secrets) now explicitly re-arm on every
  recurrence within a session instead of being treated as satisfied once a
  stage completes; retroactive "did you do X?" questions require re-checking
  evidence, not answering from memory.
- Hard-TL review before a PR now prefers delegating to a dedicated
  code-review skill, a multi-agent orchestration tool (diverse lenses, not
  identical clones), and a code-comprehension skill where available, with
  the hand-rolled single-subagent version as the standalone fallback.
- Self-improvement gains a third layer: a periodic outward-looking frontier
  scan for new AI-assisted engineering techniques/tools, proposed as a diff
  and never auto-merged, alongside the existing findings-log/distillation
  loop.

## 2.3.0

- Added the self-improvement system: per-area findings logs
  (`reference/findings/`) plus a periodic distillation pass that promotes
  durable, generalizable lessons into the actual rule text.

## 2.2.0

- Self-improvement pass from real QA findings (SCOVER session): folded
  concrete lessons about post-merge verification back into the skill's own
  rule text.

## 2.1.0

- Added the QA and production-verification area: re-verifying already-
  merged/shipped work, naming evidence tiers, logging findings.
- Added merge-conflict resolution discipline: treat a conflict as a task,
  not a keystroke.
- Added the hard-TL adversarial review step before opening a PR.

## 2.0.0

- Restructured from a single flat file into progressive disclosure
  (`SKILL.md` as index, detail in `reference/*.md`), and deepened content
  substantially.

## 1.x

- Initial engineering-workflow skill/plugin, plus the mission-decomposition
  rule (small verified steps, not parallel unverified duplication).
