# Self-improvement: how this skill gets better over time

This skill is meant to compound — every session that uses it should be able
to leave it slightly better than it found it, without turning it into an
ever-growing pile nobody reads. That requires two distinct layers, not one:

1. **Findings logs** ([reference/findings/](findings/)) — fast, cheap,
   per-session. Append a dated entry the moment something real is learned.
2. **Distillation** — slow, periodic. Read the accumulated entries, promote
   whatever is durable and generalizable into the actual rule text in
   `SKILL.md`/`reference/*.md`, then thin the log.

Skipping layer 2 is the failure mode: a log that only ever grows becomes
expensive to load and nobody re-reads it, which is worse than not logging at
all. The log is raw material; the reference docs are the actual product.
Only the reference docs should compound — the log should stay roughly
constant-sized over time, churning as entries get promoted or pruned.

## Layer 1 — logging (do this often, keep it cheap)

**When to log** — mirror real-correction discipline, not routine-work
discipline: log when a correction happened (something you did was wrong and
got fixed) or when a non-obvious pattern was confirmed a second time.
**Don't** log a routine session that went fine with no surprises — that's
noise, not signal, and it's exactly what makes logs unreadable over time.

**Where to log** — one file per Area, at
`reference/findings/<area-slug>.md` (the slug matches the Area's own
reference file, e.g. `reference/findings/qa.md` pairs with
`reference/qa-and-production-verification.md`). **Create the file on demand,
the first time that area has a real finding — don't pre-create empty logs
for areas that haven't needed one.** That's the same
no-unrequested-abstraction discipline this skill already asks of code
changes, applied to itself.

**Format per entry** — short, and load-bearing content first:

```md
## YYYY-MM-DD — one-line title

**Generalizable lesson:** the rule, stated so it survives with every
project-specific noun removed. This is the part that gets promoted later —
write it as if it already belonged in the reference doc.

**Why / what happened:** the concrete story that produced the lesson. Keep
project specifics (repo names, ticket numbers, file paths) here, not in the
lesson above — this part is optional supporting color, and it's what gets
trimmed once the lesson above is promoted.
```

Keep entries genuinely short. If a story needs paragraphs of specific
context to make sense, that's a sign it belongs in that project's own
`CLAUDE.md`/memory instead of here (see "keep this repo app-agnostic"
below) — not a sign to write more.

## Layer 2 — distillation (do this occasionally, on a trigger)

**Trigger** — any of: a log file has accumulated roughly 6-8 entries; the
same lesson shows up a second time in the same log; or you're starting new
work in an area and its log has anything unread. Whoever notices the
trigger does the distillation — it doesn't have to be the session that
wrote the entry.

**What distillation actually does:**

1. Read the log's entries.
2. For each one, decide: is this durable and generalizable enough to become
   real rule text? If yes, **edit the prose of the relevant
   `reference/<area>.md` file to fold it in** — integrate it into existing
   sentences/sections where it fits, don't just tack a new bullet onto the
   end of every list. A rule that's actually been absorbed into the prose
   reads as part of the document, not as a changelog of past incidents.
3. Once an entry is promoted, replace it in the log with a one-line pointer:
   `Promoted into <file> on <date>: <one-line summary>` — keeps a traceable
   record without keeping the full text loaded forever.
4. Entries that aren't generalizable enough to promote, but also aren't
   wrong, can stay as-is for now (they might earn promotion once a second
   instance shows up) — or get deleted if enough time has passed with no
   recurrence and they clearly won't be needed again.
5. Bump `version` in `.claude-plugin/plugin.json` (see
   [release-and-tickets.md](release-and-tickets.md)) since this changed
   the actual behavior the skill teaches, not just its logs.

This is the same "no code before a plan" and "evidence before claims"
discipline the rest of this skill teaches, applied reflexively: don't
promote a one-off anecdote into a durable rule on a single data point, and
don't claim a lesson is "learned" by the skill until it's actually in the
prose a future session will read.

## Keep this repo app-agnostic while doing all of this

This skill's whole value is being installable into any repo, in any
language, with no project-specific content (see the top-level `README.md`).
That constraint applies to the findings logs too, not just the reference
docs:

- The **generalizable lesson** in any entry must survive with every
  project-specific noun stripped out. Write it that way from the start —
  don't write the specific story first and hope it generalizes later.
- Project-specific detail (a ticket number, a repo name, a file path) is
  optional supporting color for the "why" section only. It should never be
  the load-bearing part of an entry.
- If a lesson genuinely isn't reusable outside one specific project, it
  doesn't belong in this repo at all — it belongs in that project's own
  `CLAUDE.md` or memory system instead. Don't let this skill become a
  cross-project ticket history because logging here was easier than
  logging there.

## Areas this applies to

Every Area this skill documents can grow its own findings log the same way
QA did — `reference/findings/planning-and-scope.md`,
`reference/findings/verification.md`,
`reference/findings/git-and-safety.md`,
`reference/findings/merge-conflicts.md`,
`reference/findings/release-and-tickets.md`,
`reference/findings/using-specialized-skills.md`, and
[`reference/findings/qa.md`](findings/qa.md) (already seeded). None of these
should exist until an area actually has a real, generalizable finding to
record — see "create the file on demand" above.

## This repo is a shared working directory (see git-and-safety.md)

More than one concurrent session may be logging or distilling at once.
`git fetch`/`pull` before editing a findings log or a reference doc, and
merge properly rather than overwrite if another session's entry is already
sitting there uncommitted or already pushed — see
[git-and-safety.md](git-and-safety.md)'s "Shared skill/plugin repos are
shared working directories too."
