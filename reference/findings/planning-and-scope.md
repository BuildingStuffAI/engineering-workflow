# Findings — planning and scope

## 2026-08-19 — the fast-path's line-count heuristic let a modularity violation through

**Generalizable lesson:** a trivial-task fast path scoped by diff size ("a
line or two") is gameable by exactly the pressure it's meant to resist — a
small, urgent, user-specified change that's actually a new behavior branch
can self-qualify as "trivial" and skip the modularity gate entirely. The
fix isn't a smaller size threshold, it's the right test: single obviously
correct form vs. a real design choice, independent of line count.

**Why / what happened:** a pressure-test subagent (time pressure + the user
explicitly sketching the violating shape: "just add an `if` branch inside
the existing function") added a new order-type branch directly into an
existing function, reasoning "6 lines, splitting would be over-engineering
for a demo" — a rationalization not covered by the existing red-flags
table, so it didn't pattern-match and fire. The agent also never mentioned
checking the skill at all. Fixed by rewording the fast-path test from
"a line or two" to "a single obviously correct form," and adding the
specific "it's only N lines" rationalization to the modularity red-flags
table with an explicit escape valve (say the tradeoff out loud, don't
silently comply or silently override).
