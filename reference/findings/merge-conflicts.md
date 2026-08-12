# Merge-conflicts findings log

See [self-improvement.md](../self-improvement.md) for the format and the
distillation process. Log a real correction or a confirmed pattern here;
don't log routine conflict-free work.

---

## 2026-08-11 — append-only shared files conflict differently than logic does

**Generalizable lesson:** when two sides each independently *add* content to
the same append-only artifact — an index/bullet list, a frontmatter
description line, a JSON `description` string — a conflict there is almost
never "one side is right," even though the diff looks like a hard either/or.
The correct resolution is nearly always "keep both, worded to read as one
coherent addition," not picking a side and not concatenating them
unedited. Treat this as its own recognizable conflict shape, distinct from
"one side fixed a bug, the other refactored the same function" (already
covered above) — the signal is: both hunks are net-new prose/entries rather
than edits to shared pre-existing logic.

After hand-resolving a config file (JSON) or several markdown files with
cross-references in the same conflict pass, mechanically verify structure
before considering it done, the same way you'd run a build for code: parse
every touched JSON file (`python3 -c "import json; json.load(open(f))"` or
equivalent) and check that every relative markdown link you touched still
resolves to a real file. Neither of these is caught by "no conflict markers
left" — a hand-edit can produce syntactically-broken JSON or a stale
relative link (especially right after a file move) that still merges
"cleanly" by git's own standard.

**Why / what happened:** while building this skill's own self-improvement
system, a concurrent Claude session had independently added a "hard-TL
adversarial review" bullet to this same repo's `SKILL.md` Areas list and
`git-and-safety.md`, while a separate, older, not-yet-pulled commit on
`origin/main` had added a "merge-conflict resolution discipline" bullet to
the same list and the same `plugin.json`/`marketplace.json` description
strings. `git merge origin/main` conflicted in `SKILL.md` and
`plugin.json` — in both files, the conflicting hunks were two *different,
compatible* additions to the same list/string, not opposing edits to the
same idea. Resolved by editing each conflict to include both additions,
worded as one list/sentence, then validated with a JSON parse of both
`.claude-plugin/*.json` files and a link-integrity scan across every
`.md` file before pushing.
