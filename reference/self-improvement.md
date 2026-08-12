# Self-improvement: how this skill gets better over time

This skill is meant to compound — every session that uses it should be able
to leave it slightly better than it found it, without turning it into an
ever-growing pile nobody reads. That requires three distinct layers, not
one — two that learn from *this skill's own mistakes*, and one that goes
looking *outward* so the skill doesn't just get more correct, it keeps
growing new capability:

1. **Findings logs** ([reference/findings/](findings/)) — fast, cheap,
   per-session. Append a dated entry the moment something real is learned
   from a correction or a confirmed pattern.
2. **Distillation** — slow, periodic. Read the accumulated entries, promote
   whatever is durable and generalizable into the actual rule text in
   `SKILL.md`/`reference/*.md`, then thin the log.
3. **Frontier scan** ([below](#layer-3--frontier-scan-do-this-periodically-look-outward))
   — slow, periodic, but *outward-looking* rather than reactive: actively
   search for engineering-workflow techniques, patterns, and tools that
   didn't come from a mistake in this repo at all, evaluate them, and fold
   the ones that generalize into the rule text the same way distillation
   does.

Skipping layer 2 is one failure mode: a log that only ever grows becomes
expensive to load and nobody re-reads it, which is worse than not logging at
all. The log is raw material; the reference docs are the actual product.
Only the reference docs should compound — the log should stay roughly
constant-sized over time, churning as entries get promoted or pruned.

Skipping layer 3 is the other, quieter failure mode: a skill that only ever
reacts to its own past mistakes plateaus at "doesn't repeat old errors" and
never gets meaningfully more capable. Real growth needs new input from
outside this repo's own incident history, not just a longer memory of it.

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

## Layer 3 — frontier scan (do this periodically, look outward)

**Trigger** — any of: the user asks to run a self-improvement/frontier pass;
a distillation pass (Layer 2) is already happening and it's a natural moment
to also look outward; or noticeably long real-world time has passed since
the last scan (this file has no memory of dates on its own — check
`git log -- reference/self-improvement.md` and the frontier section below for
the last recorded scan date).

**What a frontier scan actually does** — this is research, not
implementation, and it produces a proposal, never a silent edit:

1. Search outward for what's new in AI-assisted engineering workflow since
   the last scan: `WebSearch`/`WebFetch` for recent developments in agentic
   coding practice, spec-driven/plan-driven workflows, verification and
   eval techniques, prompt-engineering and skill-authoring patterns, and
   notable tools/repos in the space (e.g. other public agent-skill
   collections, Anthropic's own published engineering/agent-skill guidance,
   widely-discussed post-mortems of AI-assisted engineering failures).
   Cast wide on the query terms — "AI agent code review workflow",
   "agentic coding best practices", "claude code skills patterns",
   "spec-driven development", "LLM verification hallucination mitigation" —
   rather than one narrow search.
2. For each candidate finding, evaluate it against what this skill already
   is, not just whether it sounds good in isolation:
   - Does it generalize across languages/repos, or is it tool-specific in a
     way that would break the app-agnostic constraint below?
   - Does it fill a real gap in an existing Area, or does it duplicate a
     rule that already exists in different words?
   - Is it a genuine technique/discipline, or a one-off anecdote/marketing
     claim dressed up as a pattern? Skepticism here matters more than
     coverage — adding a bad rule is worse than missing a good one.
3. Write candidates up the same way a findings-log entry works (generic
   lesson first, source/context second) and propose them to the user as a
   diff — same commit-approval discipline as any other change to this
   shared repo (see git-and-safety.md). Never auto-merge a frontier finding
   into the rule text without the user seeing and approving the actual
   wording; "the web says so" is not evidence-before-claims, it's a
   secondhand claim that still needs the user's judgment call.
4. Record the scan itself, even when nothing new was adopted: date, what was
   searched, what was found and rejected and why. This is what lets the
   next scan pick up from here instead of re-covering the same ground —
   append it to `reference/findings/self-improvement.md` (create it on
   first use, same "create on demand" rule as any other findings log).
5. Bump `version` in `.claude-plugin/plugin.json` only if something was
   actually adopted into the rule text — a scan that found nothing worth
   adopting is not a behavior change.

**Why this is a separate layer from distillation:** Layer 2 promotes
lessons this skill already learned the hard way, inside one repo's own
history. Layer 3 exists because the field this skill describes — AI-assisted
engineering practice — keeps moving, and a skill that never looks outside
its own incident log will fall behind changes it never had a chance to learn
from firsthand (new verification techniques, new failure modes other people
already hit, new tools worth adopting). The goal is a skill that stays
current with how AI-assisted engineering is actually practiced, not just one
that never repeats its own past mistakes.

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
