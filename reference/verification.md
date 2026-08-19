# Verification and testing

## Contents
- The gate: evidence before claims
- Test before, during, and after
- Loop until verified — but bounded
- Red flags

## The gate: evidence before claims

**No completion, fix, or passing claim without a fresh verification command run
in this exchange.** "Should work now," "looks correct," or trusting a previous
run are not evidence.

Before claiming any status:
1. Identify what command actually proves the claim (test suite, build, the
   specific reproduction of the original bug).
2. Run it, fresh and in full — not a partial/cached check.
3. Read the real output: exit code, failure count, not just the last line.
4. Only then state the claim — with the evidence, not instead of it. If the
   evidence doesn't support the claim, report the actual state.

If you can't run the verification (no test exists, can't access the running
app), say that explicitly rather than implying you verified something you
didn't.

## Test before, during, and after

- **Before**: confirm the current baseline is actually green before you start.
- **During**: a new function/feature gets its test alongside it — real edge
  cases, not one happy path. A test that mocks away the exact mechanism a
  bug lives in (e.g. hardcoding the value a build step would normally
  compute) proves nothing about that bug — it would pass identically
  whether the bug were present or reintroduced. Check the test actually
  exercises the real pipeline/mechanism, not a stand-in for it.
- **After**: re-run the full suite, not just the new test. For UI/behavior
  changes, verify in the running app if at all possible. "Verified in
  isolation" ≠ "verified together" — do one combined regression pass across
  the whole change before calling it done, not just per-piece checks.

**Violating the letter of "test alongside" is violating the spirit of it.**
Writing the test in a later turn, or after saying "done," is not "during" —
it's "after," restated. If the installed TDD skill is available, dispatch it
instead of improvising this inline — it enforces the write-test-first order
this section only summarizes.

| Excuse | Reality |
|---|---|
| "I'll add tests after this stage" | After never comes as a real turn — write it alongside, same turn, before moving on. |
| "It's a simple function, doesn't need a test" | Simple functions still have a wrong-input/empty/boundary case. Test it anyway. |
| "The existing suite probably covers this" | "Probably" isn't evidence — grep for the actual test name, or write one. |
| "One happy-path test is enough" | Add the edge and error cases too, not just the golden path. |
| "I'll modularize/clean this up at the end" | Do it now, in the same function you're writing — "later" becomes a separate ask nobody made. |

## Loop until verified — but bounded

Iterating until a real check passes is a good pattern, not a bad one — but
only under three conditions:

1. **The check is real and fresh each time** — an exit code, a failing-then-
   passing test — never "looks right now."
2. **Each attempt is informed by why the last one failed**, not a blind retry.
   Retrying the same fix expecting a different result is thrashing, not
   iteration — if you don't know why it failed, that's a debugging problem to
   solve first, not another attempt to burn.
3. **There's a hard cap.** After a small number of failed attempts (three is a
   reasonable default), stop and report to the user with what you tried and
   what you learned, instead of continuing to loop.

## Red flags

Treat any of these, in your own output, as a signal to stop and actually
verify before continuing:

| Phrase / behavior | What to do instead |
|---|---|
| "Should pass now" / "should work" | Run it. |
| Expressing satisfaction before running the check ("Great, done!") | Run the check first, then report. |
| About to commit/push/PR without having run anything this turn | Run the relevant check first. |
| Trusting a subagent's/tool's own "success" report | Verify independently (diff, output, re-run). |
| "Partial check is enough" / "the linter passed" | A linter is not a compiler; a partial check is not a full one. |
| Retrying the same fix a second time with no new information | Stop — go find the actual root cause first. |
| "Already did this gate earlier in the session, this is a small follow-up" | Wrong — each new edit/branch/PR re-arms its gate independently. See SKILL.md "Gates persist for the whole mission." |
| Asked "did you do X?" (run tests, use hard-TL review, etc.) | Re-check the actual transcript/diff/git log/test output for evidence before answering. If there's no evidence, say so — don't answer from memory or assume it happened because it usually does. |
| Calling a same-context self-review "the hard-TL review" / "an adversarial review" | Not the same thing. If you didn't dispatch a genuinely separate subagent/tool with no context from your own work, you haven't run it — see git-and-safety.md's "Hard-TL adversarial review." |
