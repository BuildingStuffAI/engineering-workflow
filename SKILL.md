---
name: engineering-workflow
description: General software-engineering workflow discipline for any repo — plan before code, mission decomposition into small verified steps, one-action functions, evidence-before-claims verification, bounded loop-until-verified, destructive-action safety, secret hygiene, git/PR gating, merge-conflict resolution discipline, version-bump hygiene, file-creation approval, ticket discipline, post-merge/production QA verification with a persistent findings log, self-improvement via per-area findings logs distilled back into the skill's own rules, and using specialized skills/plugins (debugging, TDD, planning, code review, design) instead of reinventing them. Use at the start of and throughout any task that changes code behavior, in any language/codebase — a short diff ("just add one branch") is not trivial merely for being short, don't pre-judge it as too small to invoke this skill.
---

# Engineering Workflow

Default operating mode for engineering work, distilled from repeated real
corrections, not theory. A repo's own `CLAUDE.md`/`docs/` wins over this when
they conflict — this is the floor, not a ceiling.

**Tradeoff:** biases toward caution over speed. Use judgment on trivial work —
don't ceremony-gate a typo fix.

## Trivial-task fast path

Before reading anything below: is this a small, low-risk, single-file change
(a typo, a rename, a comment, an obvious one-liner fix with a single correct
answer) with no new files, no push/PR, and no completion claim about someone
else's system? If yes, **do not read any reference/*.md file** — state the
one-line plan out loud, make the edit, run/verify it if trivially checkable,
done. Loading full reference docs for a two-line fix is the ceremony this
skill explicitly tells you not to do (see planning-and-scope.md's "don't
ceremony-gate trivial work" — but you don't need to open that file to know
it).

**Line count is not the test.** Adding a new case/branch/behavior to
existing code is never trivial, even when it's two lines — it's a design
decision (see "Architecture decisions" and "Red flags — modularity" in
planning-and-scope.md), not a mechanical fix, and skips the modularity gate
if waved through here. The test is "does this have a single obviously
correct form," not "is the diff short."

Reference files are loaded **lazily, one at a time, only when their specific
gate actually fires** — never as a batch at skill-start. Opening a PR fires
git-and-safety.md; nothing else on the list needs opening yet. A completion
claim fires verification.md. Don't pre-load the rest "to be thorough."

## The two hard gates

**No code before a plan.** What's affected, blast radius, risk — right-sized
to the task. See [reference/planning-and-scope.md](reference/planning-and-scope.md).

**No completion claim without fresh verification evidence run in this
exchange.** "Should work" is not evidence. See
[reference/verification.md](reference/verification.md).

Everything below expands on these two, plus adjacent discipline.

## Gates persist for the whole mission

Finishing one stage does not retire this skill for the rest of the session.
Each trigger below re-arms its gate independently, every time it happens —
not once per mission, not "already covered earlier":

| Trigger (recurs, doesn't happen once) | Re-armed gate |
|---|---|
| Request has 2+ reasonable readings | Ask, don't guess — [planning-and-scope.md](reference/planning-and-scope.md) "Clarify before you plan" |
| A step has a real design choice (library, data model, security posture, sync-vs-async) | Name the tradeoff + recommendation in plain language before coding — [planning-and-scope.md](reference/planning-and-scope.md) "Architecture decisions" |
| Change touches auth, secrets, user input, payments, or a trust boundary | Run `/security-review` — [using-specialized-skills.md](reference/using-specialized-skills.md). Default is to always run it; self-judging the diff "too small/quick to bother" is the same silent-skip failure the hard-TL gate forbids — say so out loud and get agreement instead of deciding quietly. |
| Any code edit, however small | Tests before/during/after — [verification.md](reference/verification.md) |
| A new function/behavior is written | Its test is written in the SAME turn, not "after" — [verification.md](reference/verification.md). Prefer dispatching the installed TDD skill over improvising this inline. |
| Code could be squeezed into an existing function/file instead of a new one | Modularity check before writing — [planning-and-scope.md](reference/planning-and-scope.md) |
| About to hand-roll a process for a well-known problem (debugging, review, TDD, planning) | Search installed skills first — [using-specialized-skills.md](reference/using-specialized-skills.md) |
| Opening/pushing to a branch or PR, even a small follow-up on an already-reviewed branch | Hard-TL adversarial review — [git-and-safety.md](reference/git-and-safety.md) |
| A commit | Secrets/staged-diff check — [git-and-safety.md](reference/git-and-safety.md) |
| Any claim of completion/fix/pass | Fresh evidence gate — [verification.md](reference/verification.md) |
| A retroactive question ("did you do X?") | Re-check actual evidence before answering, never memory — [verification.md](reference/verification.md) |

Track this with an explicit `TaskCreate` checklist per mission instead of
memory: at mission start, create one task per gate expected to recur; the
next time its trigger fires, re-open that same task rather than assuming a
stage that ran once already covers what comes after it. If a gate is
genuinely unnecessary for a specific trigger, say so out loud with the
reason, in that moment — a silent skip is exactly the failure this table
exists to prevent, not a judgment call to make quietly.

## Areas

- **[Planning and scope](reference/planning-and-scope.md)** — clarify
  ambiguous requests before planning, not after; plan before code; name
  tradeoffs and a recommendation in plain language for real design
  decisions; break missions into small, independently-verified steps; read a
  reference/spec completely before implementing; keep the user in the loop
  with small increments; one function = one action; modularity and scope
  discipline (no unrequested abstractions); file-creation needs approval.
- **[Verification](reference/verification.md)** — the evidence-before-claims
  gate; test before/during/after; loop-until-verified, but bounded (real
  fresh check each time, informed retries, hard attempt cap); red flags to
  catch in your own output.
- **[Git and safety](reference/git-and-safety.md)** — destructive-action
  safety (`git status` before anything destructive, never force-push to
  shared branches); secrets/review before push; commit and push to a branch
  freely, but gate PRs and pushes-to-main as a separate explicit ask every
  time; a hard-TL adversarial review pass before opening a PR — preferring a
  dedicated code-review/multi-agent/code-comprehension tool when one exists,
  falling back to a hand-rolled skeptical subagent when none does; per-repo
  delivery conventions differ, confirm don't assume.
- **[Merge conflicts](reference/merge-conflicts.md)** — treat a conflict as a
  task, not a keystroke: plan before resolving, never blindly take "ours" or
  "theirs" on a whole file, re-verify and re-review after resolving, escalate
  when the other side's intent is unclear.
- **[Release and tickets](reference/release-and-tickets.md)** — bump version
  markers for shipped-behavior changes; a version bump is a text edit, not a
  build — don't build/ship unless asked; ticket discipline scoped to the
  whole connected change.
- **[Using specialized skills](reference/using-specialized-skills.md)** —
  check for and use an existing debugging/TDD/planning/code-review/design/
  code-comprehension/multi-agent-orchestration skill or plugin before
  hand-rolling the equivalent inline, including for the hard-TL review
  itself; close the loop (self-review, update docs, proactive context-limit
  handoff, get it right on a branch before promoting to production).
- **[QA and production verification](reference/qa-and-production-verification.md)**
  — re-verifying already-merged/shipped work: confirming merge actually
  deployed (don't assume push = live, especially with concurrency-cancelled
  deploy pipelines), re-checking the current merged tip (not each PR's own
  branch) for cross-PR collisions, running tests against the real merged
  code, a live check in the running app, naming your evidence tier, and
  logging every finding — pass or reject — to
  [reference/findings/qa.md](reference/findings/qa.md) so nothing gets
  rediscovered from scratch next time.
- **[Self-improvement](reference/self-improvement.md)** — how this skill
  compounds without bloating: log real findings cheaply per-area in
  [reference/findings/](reference/findings/), then periodically distill
  anything durable into this skill's own rule text and thin the log — keeps
  this repo app-agnostic (generalizable lesson first, project specifics
  optional) even though multiple projects feed it over time — plus a
  periodic outward-looking frontier scan (web search for new AI-assisted
  engineering techniques/tools, evaluated and proposed as a diff, never
  auto-merged) so the skill keeps gaining capability, not just avoiding
  repeat mistakes.

---

**Working if:** plans precede diffs, diffs stay scoped to the ask, nothing
destructive happens without a status check, nothing ships or merges without
an explicit go-ahead, and every completion claim has fresh evidence behind it.
