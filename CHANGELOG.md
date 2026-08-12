# Changelog

All notable changes to this skill are recorded here. Versions match
`.claude-plugin/plugin.json`.

## Unreleased

- Closed a same-context-self-review loophole in the hard-TL gate: a diff
  re-read in the same context it was written in doesn't satisfy the gate
  just because it's labeled "a hard-TL review" — only a genuinely separate
  dispatch does. Found via a paired control/test pressure test (subagents
  with and without the v2.4.0 wording, identical time-pressure scenario).

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
