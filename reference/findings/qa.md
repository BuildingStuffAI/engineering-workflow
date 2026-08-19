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

---

## 2026-08-12 — horoscope-launcher + magnifier-launcher — LAUNCH-45 (post-merge QA)

**Context:** feature ask was "hide the embedded main app from search/app-drawer
discoverability." Investigation found the launcher's own in-app search already
worked (pre-existing `AppFilter`/`hiddenApps`), but a real bug let the embedded
app leak into the **all-apps drawer grid** itself — worse than the original
search-box concern, since drawer browsing is the more obvious discovery path.
Root cause: `build.gradle`'s `launcher_component`/`launcher_entry_component`
resValues were computed inside `productFlavors { configureEach { ... } }`
using `${applicationId}`, which resolves *before* Gradle applies the debug
build type's `applicationIdSuffix ".debug"` — baking a component string
(`com.launcher.horoscope/...`) that didn't match the actual runtime package
(`com.launcher.horoscope.debug/...`), silently breaking the `ComponentName`
Set-membership check in `AppFilter.shouldShowApp()`. Fixed in both repos by
moving the resValues into the already-correct `applicationVariants.configureEach
{ variant -> ... }` block using `variant.applicationId` (the same pattern
already used there for `lawnchair_application_id`). PRs: horoscope-launcher
#4, magnifier-launcher #1, both merged to `main` after a hard-TL review pass
(`git merge-tree` clean, diff scoped to exactly the intended lines, no
secrets). Jira: LAUNCH-45.

**First pass of "post-merge verification" only rebuilt the merged tip and
checked `BUILD SUCCESSFUL`** — the user directly asked "did you use the
skill's post-deploy method?" and the honest answer was no: a green build is
not the same as the fix actually working live. Did a proper second pass:

- **magnifier-launcher — PASS.** Pulled merged `main`, `grep` confirmed the
  two resValue lines are in the correct block, fresh (`uninstall` +
  `install`, not `-r`) debug build installed on emulator, set as home,
  swiped up to the all-apps drawer: no "Magnifier (Debug)" self-tile.
  Home-screen icon still opens the app normally. Live-confirmed, not just
  code-read.
- **horoscope-launcher — PASS.** Built the merged-`main` debug APK, installed
  fresh on a *second*, separate emulator (avoids the cross-app noise a
  shared device produces once multiple sibling forks are installed
  alongside each other) and confirmed live: all-apps drawer shows no
  "AstroCircle" self-tile, in-app search for "astrocircle" returns no app
  result, home-screen icon still present and functional.

**Generalizable rule (the actual payoff of this entry):** "rebuild from
merged main and confirm `BUILD SUCCESSFUL`" is a *build* verification, not a
*behavior* verification — it proves the code compiles, not that the fix does
what it claims at runtime. For any fix whose evidence was originally a live
UI/emulator check pre-merge, the equivalent post-merge QA step is to redo
that same live check against the merged artifact, not just rebuild it. Name
this distinction explicitly in the verdict ("build verified" vs "live
verified") so the gap doesn't need to be surfaced by the user asking a
pointed question — same lesson as the SCOVER entry above, recurring in a
different repo family, which is itself the signal this belongs in the
skill's rule text rather than staying a one-off log entry.

**Also relevant to future sessions in this repo family:** reusing one shared
emulator across horoscope-launcher/magnifier-launcher/eco-launcher/
flashlight-launcher for sequential testing causes real cross-app noise in
drawer screenshots (each app's icon shows up in every other app's drawer
once co-installed, since they're genuinely separate installed packages on
the same device) — harmless to interpret once you know why, but worth
booting a second emulator for true parallel verification instead of
serially juggling one, especially once more than 2 of these forks need
checking in the same session.

### Follow-up: the same fix ported to eco-launcher + flashlight-launcher, and a self-review vs. hard-TL gap caught mid-session

The same `applicationIdSuffix`/`launcher_component` bug turned out to affect
eco-launcher and flashlight-launcher too (their `LawnchairLauncher` intent-filter
explicitly declares `category.LAUNCHER`, unlike horoscope/magnifier's, which is
why only these two actually showed the self-listing symptom live). Same fix
pattern applied and merged: eco-launcher PR #157, flashlight-launcher PR #35.

**Process gap caught by the user mid-session, twice, worth internalizing:**

1. After merging all 4 PRs (horoscope #4, magnifier #1, flashlight #35,
   eco #157), the "hard-TL adversarial review" step described in each
   agent's report was **not** what `git-and-safety.md` actually prescribes —
   it was the same PR-authoring agent reviewing its own diff in the same
   context, not "a fresh subagent with no context from your work." The
   `git merge-tree` conflict check was real; the adversarial-review claim
   was not. The user caught this by asking directly ("did you use the Hard
   TL and verification of conflicts before merging?") rather than it being
   self-caught.
2. Ran genuine independent reviews after the fact (4 fresh subagents, one
   per repo, given only the diff + neutral root-cause text, no framing that
   it was "already verified") — all 4 converged on **zero MUST-FIX
   correctness bugs** (each independently confirmed `variant.applicationId`
   resolves post-suffix, no sibling `${applicationId}`-in-flavor-block
   bugs elsewhere in any of the 4 repos, no stale-base drift, no dangling
   Groovy) but **all 4 independently found the same real gap**: none of
   the 4 repos' `AppFilterTest.kt` exercises the actual Gradle `resValue`
   pipeline — it mocks `R.array.filtered_components` with a hardcoded
   string, so it would pass identically whether this exact bug were present
   or reintroduced. Not yet fixed (test-writing is new scope, flagged to
   the user rather than assumed).

**Generalizable rule:** "I performed a hard-TL adversarial review" is a
claim that needs the same evidence-tier discipline as any other verdict —
if the review was done by the same agent/context that wrote the fix, say so
plainly ("self-review passed, independent review not yet done") rather than
labeling it "hard-TL," which specifically means a separate, context-free,
skeptical pass per this skill's own git-and-safety.md. Four independent
reviews landing on the identical non-blocking gap (rather than four
different nitpicks, or four rubber-stamps) is itself evidence the
independent-pass requirement is doing real work — a shared-context
self-review would likely not have surfaced "no test exercises the real
resValue pipeline" at all, since the author already believes the manual
emulator verification is sufficient proof.

**Final live QA, all 4 repos — PASS.** Fresh install + drawer check on
real emulators for all four: horoscope-launcher, magnifier-launcher,
eco-launcher, flashlight-launcher all show no self-listing tile post-merge.
One real snag hit and resolved along the way: eco-launcher's drawer
initially showed an "Eco Launcher (Debug)" tile that looked like a fix
regression, but turned out to be a **separate, pre-existing package**
(`com.eco.launcher.debug`, the `github` flavor, an unrelated leftover
install from a prior session on the same AVD) — not the freshly-built
`com.launcher.eco.debug` (`play` flavor) with the fix. Confirmed by
`aapt2 dump resources` on the actual built APK (correct value baked in)
before concluding the tile was unrelated stale state, not a regression —
i.e., verified the artifact directly rather than trusting the visual
symptom alone. **Generalizable rule:** on a long-lived shared emulator/AVD,
`pm list packages | grep <name>` before diagnosing a surprising drawer/search
result — a differently-flavored package with the same or similar display
name can coexist and produce a misleading symptom that looks exactly like
a regression.

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

---

## 2026-08-12 — sherlock-plus — SCOVER-119 (pre-fix QA: does an existing PR already cover it?)

**Context:** before planning SCOVER-119 ("don't show a Search History profile
image before a candidate is selected/confirmed"), asked to check whether
commit `759e10c` ("Update /tmp history label when identity locks.", PR #119
"Update /tmp history when identity locks", merged 2026-08-01, title
coincidentally shares the "119" number with the *different* Jira ticket
SCOVER-119 — a numeric collision, not the same 119) already fixes it.

**Deploy confirmation:** `759e10c` is several commits behind `upstream/main`
tip, and every `deploy-prod.yml` run since (through 2026-08-12) succeeded
with no `cancelled` runs in between — it is live in production, not just
merged.

**Code-level verdict: REJECT — that commit is the likely root cause for
identifier-based searches, not a fix.** Its own commit message says the
intent was to "show the person name as soon as it is known" for
email/phone/LinkedIn searches — i.e. deliberately optimistic, pre-
confirmation display. It added two unconditional `patchSessionInListCache`
calls in `useInlineInvestigation.ts` (`applyCompletion` ~L168-175, and the
poll-loop hydration path ~L520-527) that patch `persona_photo_url` via
`resolveWizardSessionPersonaPatch` with **no phase gate at all** — not even
the `shouldPersistWizardSessionPersona` gate that exists (and is itself
incomplete: it excludes `"name"`/`"disambiguate"` phases but not
`"identity_preview"`) for the backend-persistence branch a few lines below.
Same unconditional-write shape independently confirmed in
`useInvestigationWizard.ts:153` (`syncWizardSidebarPersona`), which predates
this commit. Read directly off the current merged/deployed tip, not a
stale branch.

**Live-UI check (platform.scover.ai, logged in as the real account):**
attempted 2 live reproductions, evidence tier "live-confirmed, inconclusive
for this specific bug" — neither hit the exact race window:
1. Fresh name search ("Michael Johnson", Reconnecting purpose) → 2
   disambiguation candidates, **both rendered with initials-only avatars,
   no photo data available for either** — correctly shows no premature
   photo, but only because this data source had no `photo_url` to leak in
   the first place, not because a gate stopped it. Took longer than
   expected to resolve (~20s+) — unrelated to this bug, not investigated
   further.
2. Resumed an existing unconfirmed disambiguation (`ido-avital` LinkedIn
   search, 2 candidates, also initials-only avatars) and picked one — the
   picked person turned out to match an already-fully-resolved prior
   session ("Ido Avital", confirmed yesterday) and the app reused that
   session outright, jumping straight to the finished profile. No
   intermediate "photo appears before lock" moment was observable because
   there was no fresh intermediate state to observe.

**Gap, stated explicitly:** the bug needs a case where
`artifacts.structured.identity.photo_url` is populated with a *real,
possibly-wrong* photo during `disambiguate`/`identity_preview`, before
`wizard_state.locked_identity` is set — most plausible for a fresh
identifier search (email/phone/LinkedIn) that auto-continues past
disambiguation via `shouldAutoContinueIdentityPreview` without stopping to
show initials-only candidate cards. Did not force a third fresh search to
chase this, since each attempt consumes a real search credit
(89/100 on this account) and the code-level evidence is already strong
enough to proceed to planning — flagging this as the live-repro angle for
whoever verifies the eventual fix.

**Suggested fix direction (not applied, this was a QA pass only):** gate
`persona_photo_url` (and `persona_name`, same patch object) in the *local*
list-cache write itself at both call sites, using the same or a stricter
"confirmed" check (`wizard_state.locked_identity` present, not just
phase-name exclusion) — and separately fix `shouldPersistWizardSessionPersona`
to also exclude `"identity_preview"`, since that phase is explicitly
commented elsewhere in the code as "confirm the match before full
enrichment."

**Generalizable rule:** a ticket number matching a PR title's number by
coincidence (Jira "SCOVER-119" vs GitHub "PR #119") is a trap — confirm by
reading the actual commit diff and intent, not by pattern-matching numbers.
Also: when a live click-through can't hit the exact bug window after a
reasonable attempt, log the gap and the reasoning for stopping (credit
cost, code evidence already sufficient) rather than either quietly
declaring PASS or burning unbounded budget chasing a live repro.
   edited by more than one concurrent Claude session — mid-task, a peer
   session had already made local, uncommitted edits to `git-and-safety.md`
   in this exact directory, and `git push` was later rejected because a
   different, older commit had landed on `origin/main` in the meantime.
   Both were resolved via a real 3-way merge (not overwritten), see
   "Shared skill/plugin repos are shared working directories too" in
   `git-and-safety.md`.

---

## 2026-08-18 — horoscope-launcher — LAUNCH-56 (post-merge QA: emulator PASS, real device REJECT)

**Context:** PR #19 (`ea0c5e714c` / `7803a94795`) merged a min-height fix for the AstroCircle tap-ripple (`PhoneFrame` `heightIn(min = 260.dp, max = 300.dp)`). QA reports the broken-circle still happens on a real device; emulator looks fine. Ticket still **QA**. Original Jira screenshots are from Play package `com.launcher.horoscope` on a physical phone.

**Deploy/merge confirmation:** the fix *is* on `origin/main` tip `3a111334d5` (ancestor check of `7803a94795`). This is an Android Play-internal app, not a web deploy — testers only see it after an AAB/APK install. Connected emulators were on versionName 1.1.38 / 1080x2400 and 1440x3120, font scale 1.0 (plenty of vertical space).

**Cross-PR collision:** none with LAUNCH-54 (different files). Complete Later already uses v2 `PhoneIllustration`; first-launch uses V2; **retry loop still uses legacy `DefaultHomeOnboarding`**, which never got the min-height change.

**Verdict: REJECT (code + environment comparison; not live-confirmed on a real device).** Root cause is still `.clip(RoundedCornerShape(24.dp))` on `PhoneFrame`. When the pager/sheet is shorter than the column + 64.dp ring (nav bar, cutout, font scale, compact phone), the clip slices the circle into two arcs. Min-height only helps when the parent actually has ≥260.dp — tall emulators do; many real phones do not. PR #19 already documented remaining clip at 2x font.

**Suggested fix (not applied in the QA pass):** stop clipping phone-frame *content*; keep rounded chrome via `background`/`border` only; size with `wrapContentHeight()` + `heightIn(max = 300.dp)`; reuse the same `PhoneIllustration` on the retry path. Plan: `docs/superpowers/plans/2026-08-18-launch-56-real-device-ripple.md`. Branch: `fix/launch-56-real-device-ripple`.

**Live-UI gap, stated:** no physical device attached; `OnboardingV2Activity` is `exported=false` so adb could not deep-link page 3. Real-device confirmation stays with QA after the follow-up build.

**Generalizable rule:** an emulator-only visual fix that adds `minHeight` against a *clipping* parent is not a device fix. If the bug is “shape cut into two arcs,” the clip is the cause; min-size only masks it on large viewports. Always name emulator vs device as different evidence tiers, and check leftover duplicate UI paths (v1 vs v2) that QA can still hit.
