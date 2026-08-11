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

---

## 2026-08-11 — sherlock-plus — SCOVER-90, 92, 52, 124 (post-merge production QA)

**Context:** 4 tickets' PRs (#183 SCOVER-92, #184 SCOVER-90, #185 SCOVER-124,
#186 SCOVER-52) merged into `upstream/main` within ~15 seconds of each other.
Asked to verify all 4 actually work on production, not just that each PR's
own tests passed.

### Process-level finding (generalizable — read this even if you don't work on sherlock-plus)

The deploy workflow triggers on push-to-`main` with
`concurrency: {group: deploy-platform-prod, cancel-in-progress: true}`. The
rapid merge burst caused the deploy runs for #183/#184/#185 to be
**cancelled** — only the run triggered by the last merge (#186) completed
successfully. This was not a failure: because merges are sequential on
`main`, the successful run's `headSha` was `main`'s tip *after* all 4
merges, so it deployed all 4 changes. Confirmed via
`git merge-base --is-ancestor <each-commit> <deployed-sha>` plus the live
asset's `Last-Modified` header matching the deploy completion timestamp.

**Generalizable rule:** with `cancel-in-progress: true` deploy pipelines, a
burst of rapid merges will show `cancelled` runs for everything but the
last — don't read `cancelled` as "that change didn't deploy," and don't
assume "merged" means "live" without checking the deploy run + confirming
the deployed SHA actually is a descendant of your commit. **The real risk**
this pattern creates: if the *last* merge in a burst had failed its own
build (unrelated to the earlier merges), every earlier PR's changes would
also silently not be live, with no failure signal on those earlier PRs'
own CI. Worth a process fix (serialize deploys, or alert on any cancelled
run within a merge burst) if this recurs — flagged to the team, not fixed
in this pass (infra change, out of scope for a QA verification session).

### Per-ticket verdicts

- **SCOVER-90** (Gallery collapse chevron inert in platform mode, PR #184)
  — **PASS**. Fix (`ProfileMediaGallery`'s `open`/`onOpenChange` no longer
  hardcoded by an `isPlatform` ternary) confirmed present on `upstream/main`
  tip via `git show`, confirmed no CSS conflict (no gallery-related CSS
  rule exists anywhere in the stylesheet, so nothing else could re-break
  it), and confirmed by **actually running** the new regression test
  (`unifiedProfilePlatformVariant.test.tsx`, includes a chevron
  toggle-collapse-expand assertion) against the real merged tip in a
  disposable worktree — 9/9 pass. **Live-UI gap closed in a follow-up pass
  the same day**: the first two profiles checked (Tim Cook, Adam Agensky) had
  no gallery (renders only when `galleryItems.length > 0`); used the `find`
  tool to search the accessibility tree for a "Gallery" heading across
  history entries rather than scrolling each one blind, found one on a real
  profile, and clicked the chevron closed then open on production — photos
  actually unmounted and remounted both directions, matching the regression
  test's assertion. **Lesson**: when a UI element only appears
  conditionally, don't give up after 1-2 profiles come up empty — use an
  accessibility-tree search (`find`/`read_page`) to locate a real instance
  instead of manually sampling, especially when live-clicking is the whole
  point of the check.

- **SCOVER-92** (duplicate "Category" placeholder in idle preview, PR #183)
  — **PASS**. Root cause was a hardcoded label array
  (`DIM_CARD_LABELS`) that used `"Category"` twice; fixed to the correct
  4 unique labels. Confirmed via `git show upstream/main:...`, confirmed
  no other file in the tree still references the old placeholder text
  (`git grep`), and **live-confirmed visually** on production
  (platform.scover.ai homepage, logged in as the real account) — the
  "Here's what it'll look like" preview shows 4 distinct labels (Identity &
  assets / Professional / Personal / Behavioral), no duplicate. Existing
  test has an explicit `Set` uniqueness assertion, so a future regression
  of this exact shape would fail loudly.

- **SCOVER-124** (AI profile brief light-mode contrast + collapse/close,
  PRs #180 + #185) — **PASS**. Contrast fix (`.platform-profile-brief-body`
  color rules) confirmed intact and correctly winning its CSS cascade
  against a same-specificity Tailwind Typography rule (source-order
  analysis, not assumption). Collapse-state reset on session change
  (`briefDismissed` reset via `useEffect` keyed on `sessionId`) traced and
  looks correct. **Live-confirmed on real production**: generated an actual
  AI brief on a real profile (Tim Cook) in the live app, confirmed
  readable light-mode contrast, and confirmed the collapse chevron
  actually toggles the panel closed/open both directions. Non-blocking gap
  noted: no test exists at the `PlatformProfilePane` level asserting
  `briefDismissed` resets on `sessionId` change (only lower-level
  `ProfileBriefInline` unit tests exist) — worth a follow-up ticket, not a
  blocker.

- **SCOVER-52** (mobile-responsiveness gaps, PR #186) — **PASS**. This PR's
  own author had already run 2 rounds of adversarial review pre-merge
  (see the hard-TL pattern this repo also documents). This QA pass added an
  independent, fresh check specifically for interaction with the other 3
  same-burst PRs: confirmed via the real merge-result diff (not
  branch-vs-base, which produces false-positive "deletions" — diff the
  actual merge commit) that #186's CSS additions and #185's CSS additions
  land in non-overlapping line ranges with zero duplicate-selector
  conflicts. **Byproduct finding, pre-existing, not a regression from any
  of these 4 PRs**: `.platform-search-subtitle` has a second, unconditional
  CSS rule later in the same stylesheet file that would silently override
  any *font-size*-based mobile media-query rule for that same selector
  (later same-specificity rule wins). PR #186's specific fix used
  `display: none`, which happens to not collide with that later rule (it
  never sets `display`), so this PR is unaffected — but the underlying
  cascade footgun is real and will bite the next person who edits that
  selector's mobile behavior via font-size. **Suggested fix** (not applied
  — flagging only, per this log's scope): consolidate the two
  `.platform-search-subtitle` rule blocks into one, or move the later
  unconditional rule earlor in the file, so source order matches intent.

**Cross-ticket confirmation**: no ticket's fix depends on, or was broken
by, any other ticket's fix in this batch — all 4 are independently correct
and independently deployed.

### Process retrospective (why this session changed the skill itself)

Two process gaps surfaced, both now folded into
[qa-and-production-verification.md](../qa-and-production-verification.md) and
[git-and-safety.md](../git-and-safety.md) rather than left as one-off learning:

1. The first full QA report reported "PASS" for all 4 items, but 2 of the 4
   (SCOVER-90, SCOVER-52) actually only had code-read + executed-test
   evidence, not a live click-through — the user had to explicitly ask
   "did you also run a UI visual test?" to surface that gap. A verdict
   should name its evidence tier so that question is never necessary — see
   "Name your evidence tier" in the process doc.
2. Separately, this repo (the skill itself) is a shared working directory
   edited by more than one concurrent Claude session — mid-task, a peer
   session had already made local, uncommitted edits to `git-and-safety.md`
   in this exact directory, and `git push` was later rejected because a
   different, older commit had landed on `origin/main` in the meantime.
   Both were resolved via a real 3-way merge (not overwritten), see
   "Shared skill/plugin repos are shared working directories too" in
   `git-and-safety.md`.
