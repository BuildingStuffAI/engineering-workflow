# Findings — self-improvement

## 2026-08-12 — first frontier scan (test run, verifying the layer itself works)

Searched: agentic coding best practices 2026; spec-driven development AI
coding workflow; LLM verification/hallucination mitigation for coding
agents; Claude Code skill-authoring patterns; AI agent code review workflow
(incl. a specific follow-up on Cursor BugBot's multi-pass review claim);
Anthropic's own context-engineering guidance; AI coding agent postmortems
2026; context engineering/compaction for long-running agents; AI code
review failure-mode checklists; acceptance-criteria/definition-of-done for
agent-executed work.

Adopted: none yet — this scan was run to test whether the frontier-scan
layer itself produces real research or rubber-stamped "nothing found."
Verdict: real — it ran actual distinct searches, rejected several
candidates as duplicate/wrong-layer/marketing fluff with stated reasons,
and converged on three candidates from independent, non-vendor sources.
Proposals are sitting with the user for the adopt/reject call the layer
requires before any reference-doc edit; this entry itself is the "record
the scan even if nothing was adopted yet" requirement.

Rejected candidates and why: multi-agent hallucination-mitigation/debate
pipelines (duplicate of existing hard-TL + diverse-lens pattern); context
engineering/compaction internals (agent-architecture advice, not workflow
discipline — wrong layer for this skill); skill-authoring best practices
(meta-advice for writing skills, not workflow content); vendor comparison
roundups and a paywalled teaser article (marketing fluff, no extractable
technique); wholesale spec-driven-development tooling adoption (product-
specific, would break the app-agnostic constraint — the generalizable
kernel was extracted separately, see proposals).

Promoted into reference/git-and-safety.md and reference/planning-and-scope.md
on 2026-08-12 (v2.5.0): all three proposals adopted — (1) review-pass
order/framing variance + convergent-finding weighting, (2) destructive-action
containment for the during/after moment, (3) acceptance criteria as
agent-checkable commands written during planning.
