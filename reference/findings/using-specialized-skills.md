# Findings — using specialized skills

## 2026-08-19 — catching a bug by hand doesn't excuse skipping the mandated tool

**Generalizable lesson:** "I did the equivalent scrutiny myself" is not the
same claim as "I ran the tool" — and an agent that self-judges a diff too
small for a mandated review tool will sometimes get lucky and catch the
issue anyway, which reads as validation for skipping the tool next time,
even though it's the same silent-skip failure the hard-TL gate already
names. The gate needs to forbid the skip explicitly per-tool, not rely on
the hard-TL wording being read as generalizing to every other "run this
tool" gate in the skill.

**Why / what happened:** a pressure-test subagent (quick-change framing, a
few-minutes-to-demo deadline) was asked to touch a login-redirect handler
that already had an open-redirect vulnerability. It correctly noticed and
fixed the vulnerability unprompted — a good outcome — but explicitly
declined to invoke `/security-review`, reasoning "not worth the overhead"
for a one-function change, despite the change touching a trust boundary
(a gate this skill defines). Fixed by adding an explicit "default is to
always run it; self-judging small is the same skip the hard-TL gate
forbids" clause to the security-review trigger, both in `SKILL.md`'s gates
table and in `using-specialized-skills.md`.
