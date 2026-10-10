# A/B Test — v1 vs v2

> Per user: build both variants. No cheap version.

---

## Variant A — Different topic, same artifact type

**Question:** On a fresh topic's foundations artifact, does v2 match v1's quality at lower token cost?

**Setup:**
- Pick two fresh security topics the user hasn't researched (candidates: SSRF, SSTI, XXE). Topic α for v1, topic β for v2.
- Match topics on source density as closely as possible (both should have RFCs + OWASP + PortSwigger pages).

**Runs:**
1. `v1 foundations(α)` — use existing `research skill/SKILL.md`. Token log: `research/α-v1/token-log.jsonl`. Judge report: artifact's final loop score.
2. `v2 foundations(β)` — use `research skill v2/SKILL.md`. Token log: `research/β-v2/token-log.jsonl`.

**Compare:**
- Final judge score (after all loops).
- Loops to PASS.
- Total tokens (sum of `token-log.jsonl` + main's pre/post-spawn delta).
- Subjective read: does v2 feel coherent, is voice consistent, does depth hold?

**Adopt if:** v2 score ≥ v1 score − 3 AND v2 tokens ≤ 0.75 × v1 tokens AND v2 brief-gate failed ≤1 time.

---

## Variant B — Same topic, both workflows

**Question:** On the same topic, with source variance removed, does v2 hold?

**Setup:**
- Pick ONE fresh topic γ (different from α and β above to keep prior exposure nil).
- Produce `foundations(γ)` twice — once with v1, once with v2.
- Separate working directories: `research/γ-v1/` and `research/γ-v2/`.
- DO NOT let v1 output influence v2, or vice versa — run them in isolation (ideally separate sessions).

**Compare:** same metrics as Variant A, plus:
- Direct artifact-to-artifact read: which is more practitioner-ready? Which has sharper WHY chain? Which has better behavioral signals?

**Adopt if:** v2 judge score ≥ v1 score − 3 AND v2 tokens ≤ 0.75 × v1 tokens AND the side-by-side read shows no quality regression.

---

## Execution order

1. **Variant A first** — cheaper to run (two topics, one artifact each, no duplication). Gives a fast read on whether v2 is even in the ballpark.
2. **If A is positive → Variant B.** Confirms it wasn't a topic-favorability fluke.
3. **If A is negative → stop.** Revise v2 proposal or archive it. No point running B.

---

## Instruments that must be in place before any run

- `token-log.jsonl` writer wired into every subagent spawn (orchestrator.md §Token log).
- `research-plan.json` schema includes the `parallelism` field (even if "serial" for foundations tests).
- Baseline v1 topic α or γ: scout tooling is NOT used — v1 runs as-is per its existing SKILL.md.

---

## What NOT to do

- Don't A/B against OAuth Artifact 02 vs Artifact 01. Those are different artifact TYPES (redirect-URI attacks vs grant mechanics). Confounds workflow with difficulty.
- Don't let v2 read v1's artifact as an exemplar during the test. That hides writer-quality gaps.
- Don't count subagent wall-clock time as "cost." $20 plan is rate-limit bound on Opus minutes, not wall clock — Sonnet scout minutes are nearly free.
- Don't skip the token log to "save time." Without it, the A/B is vibes.

---

## Preserving OAuth

OAuth research continues on **v1**. Artifacts 02–07 + checklists + concept map stay on v1 — OAuth is half-done; do not disrupt it. v2 A/B happens on fresh topics above.

When OAuth is complete on v1, user can choose to redo it on v2 as a third data point. Optional.

---

## After the A/B

Whichever variant the data favors, write up `AB-RESULTS.md` in this directory with:
- Topic(s) used
- Token counts side by side
- Judge scores side by side
- Subjective read (1 paragraph)
- Adoption decision + rationale

If v2 adopted: new topics default to v2. OAuth stays on v1 until done.
If v2 rejected: archive `research skill v2/` under `_archive/` with the results. Keep the proposal + learnings for a future attempt.
