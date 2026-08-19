# QA findings log

A running, append-only record of production/post-merge QA passes — what was
checked, the verdict, the evidence, and any suggested fix. Read the relevant
entries before starting a new QA pass on the same repo/area (a documented
PASS with its reasoning is what lets you skip re-deriving it from scratch;
a documented REJECT-with-fix is what lets the next person not repeat the
same mistake). See [qa-and-production-verification.md](../qa-and-production-verification.md)
for the process this log supports.

**Format per entry:** date, repo, tickets/PRs, verdict per item, evidence,
suggested fix (if any), and anything generalizable beyond the specific repo
— that generalizable part is the real payoff of keeping this log.

**Keep entries short.** Aim for the generalizable lesson in 2-4 lines and the
supporting "why" in another handful — not a full narrative of every attempt
made. A single entry running past ~30 lines is a sign to distill it into the
reference doc immediately rather than let it sit; don't wait for the
"6-8 entries" trigger in [self-improvement.md](../self-improvement.md) if
one entry alone is already that expensive to load.

---

## Promoted (distilled into reference docs, kept here as pointers only)

- **2026-08-11 — sherlock-plus, SCOVER-90/92/52/124**: concurrency-cancelled
  deploy pattern, diff-the-real-merge-commit (not branch-vs-base), and
  conditionally-rendered-element search-don't-sample. Promoted into
  [qa-and-production-verification.md](../qa-and-production-verification.md)
  ("Deploy confirmation", step 2, step 4) prior to this distillation pass.
- **2026-08-12 — horoscope-launcher + magnifier-launcher, LAUNCH-45**:
  self-review mislabeled as hard-TL review (promoted into
  [git-and-safety.md](../git-and-safety.md) in v2.5.0); a test that mocks
  away the exact mechanism a bug lives in proves nothing about that bug
  (promoted into [verification.md](../verification.md) "Test before,
  during, and after" on 2026-08-19).
- **2026-08-12 — sherlock-plus, SCOVER-119**: a ticket ID coincidentally
  matching a PR number is a trap — confirm by diff/intent, not by
  pattern-matching numbers. Promoted into
  [qa-and-production-verification.md](../qa-and-production-verification.md)
  step 1 on 2026-08-19.
- **2026-08-12/2026-08-18 — recurring across launcher family**: verify the
  actual installed/built artifact before diagnosing a surprising result as
  a regression (stale/duplicate package, cached build) — seen twice
  independently, promoted into
  [qa-and-production-verification.md](../qa-and-production-verification.md)
  step 2 on 2026-08-19.
- **2026-08-18 — horoscope-launcher, LAUNCH-56**: emulator and real device
  are different evidence tiers for layout/clipping/rendering bugs; check
  for leftover duplicate UI paths (v1 vs v2) a fix might have missed.
  Promoted into
  [qa-and-production-verification.md](../qa-and-production-verification.md)
  "Name your evidence tier" and step 2 on 2026-08-19.

## Open follow-ups (project-specific, not yet fixed — track via that
## project's own ticket/memory system, not this cross-project log)

- sherlock-plus: `.platform-search-subtitle` has two CSS rule blocks in the
  same file where the later unconditional one silently wins for any future
  font-size-based mobile rule — not a regression from SCOVER-52, but a live
  footgun for the next person who edits that selector.
- sherlock-plus SCOVER-119: fix not yet applied — gate `persona_photo_url`/
  `persona_name` in the local list-cache write at both call sites in
  `useInlineInvestigation.ts`, and add `"identity_preview"` to
  `shouldPersistWizardSessionPersona`'s exclusion list.
- horoscope/magnifier/eco/flashlight-launcher: none of the 4 repos'
  `AppFilterTest.kt` exercises the real Gradle `resValue` pipeline (mocks
  the computed value directly) — wouldn't catch a regression of the
  LAUNCH-45 bug shape if reintroduced.
- horoscope-launcher LAUNCH-56: real fix not yet applied — stop clipping
  `PhoneFrame` content (`.clip(RoundedCornerShape)`), keep rounded chrome via
  `background`/`border` only, size with `wrapContentHeight()` +
  `heightIn(max = 300.dp)`, and port the fix to the legacy retry-loop path
  (`DefaultHomeOnboarding`) which never got it. Branch:
  `fix/launch-56-real-device-ripple`.
