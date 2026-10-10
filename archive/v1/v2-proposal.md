# Research Mastery — v2 Proposal

> Goal: cut per-artifact token cost ~50% without degrading output on Opus. Keep v1 intact as fallback.

---

## 1. Diagnosis — where v1 burns tokens

| Cost center | v1 behavior | Problem |
|---|---|---|
| Raw-source reads | Main context reads full RFCs + raw MDs | A single RFC can be 10–40k tokens; sits in context for rest of run |
| Inline Tier-1 research | `web_search` + `web_fetch` dump into main | By artifact 05 the session has 8 artifacts of search noise |
| Self-judge in main | Judge reasoning happens in same session | Judge's re-reads of artifact double its cost |
| Full rewrites on refine | Refine regenerates whole artifact | Loop 2 = ~2x loop 1 tokens, even for small diffs |
| Always ≥2 loops | Practice defaults to 2+ loops | Artifacts that score 85 on loop 1 still get a loop 2 |
| Sequential artifacts | 00 → 01 → ... → 07 strictly serial | Independent artifacts (02-07) can't parallelize |

**v1 strength to preserve:** single-writer voice on Opus produces coherent artifacts. Parallelism risks drift — mitigate, don't break.

---

## 2. v2 architecture — Orchestrator + three subagent roles

Based on Anthropic's multi-agent patterns: **Orchestrator-Subagent** for bounded tasks, **Generator-Verifier** for judge. Main context becomes a thin coordinator that holds plan state + exemplars + brief summaries — never raw sources or full artifacts in working memory.

**Orchestrator cost — named, not hidden.** Main carries, for the full topic run: `research-plan.json` (~1–3k tokens), one exemplar artifact (artifact 00 final, ~6–10k tokens), and per-artifact brief summaries (~200 tokens × N artifacts). For a 10-artifact run that's roughly 10–15k tokens of sticky state. The brief-pointer discipline in §2.4 is what keeps it from growing — if it does grow past ~25k, the run should be split into two sessions at the 00/01-done boundary. The orchestrator-bottleneck risk (info lost in handoffs) is real for findings discovered *by* a subagent that another subagent would need — mitigated because the only cross-subagent flow here is scout→writer, and the brief IS that handoff, written to disk, readable by both.

```
┌─────────────────────────────────────────┐
│   Main (Opus) — Orchestrator            │
│   Holds: research-plan.json             │
│          exemplar artifact (00 final)   │
│          brief summaries                │
└──┬────────────┬────────────┬────────────┘
   │            │            │
   ▼            ▼            ▼
 Scout        Writer        Judge
 (sonnet)     (opus)        (sonnet)
 reads raw    writes the    reads artifact
 sources →    artifact      cold → scores
 returns      from brief    + MUST list
 brief.md                   → returns diff list
```

### 2.1 Scout subagent — "Research brief producer"

**Input:** artifact spec (from `research-plan.json`) + list of raw sources + Tier-1 queries to run.

**Output:** `artifacts/briefs/{id}-brief.md` — a primary brief (target ≤800 words) PLUS, when the source is dense, one or more `{id}-brief-appendix-N.md` files holding verbatim HTTP traces, RFC quotes, and canonical examples that don't fit the budget but the writer will need. The primary brief indexes appendices by anchor. Writer reads brief first, appendices on demand.

**Brief schema (enforced):**
- Facts: each with quote + source URL + section/line anchor
- HTTP exchanges: request + success + at least one blocked/error case per technique
- WHY-at-parser-level notes extracted verbatim from RFCs (not scout-paraphrased — paraphrase loses the precision the writer needs)
- Known-real attributions with dates
- `scout_confidence: high | partial | low` per must_include item
- `source_coverage: complete | partial` — explicit if any RFC section was skipped for budget

**Fidelity protocol (the 800-word cap must not cost depth):**
- Scout MUST NOT paraphrase RFC MUSTs/SHOULDs or parser-level mechanics. Quote verbatim.
- If a must_include item can't be covered without exceeding 800 words, scout puts the overflow in an appendix file, NOT drops it.
- Before writer spawns, main runs a **brief-gate check** (one Sonnet call, ~300 tokens): reads brief + must_include + scout_confidence, emits PASS/FAIL. On FAIL, **preferred path** = re-spawn scout scoped to the gap list only (scout re-reads the specific RFC section, not the whole file). **Last-resort fallback** = writer reads the raw source, but ONLY the anchor range the gap points to (e.g., RFC 6749 §4.1, not the full RFC). Full raw-source reads into main context are forbidden — that is the pathology §1 diagnosed. Escalation path is logged per artifact so we can see if scout is systematically under-delivering.

**Model:** Sonnet (extraction, not creative synthesis).

**Token win hypothesis (unverified until §10 measures):** raw RFCs don't enter main context. Cost traded: scout spawn + brief-gate. Net expected to be positive on dense sources (RFCs), negligible on thin ones (blog posts) — exactly the pattern v1 is weakest on.

### 2.2 Writer subagent — "Artifact producer"

**Input:** artifact spec + brief.md + exemplar (artifact 00 final, for voice) + writer.md rules.

**Output:** full artifact `.md` written to disk.

**Model:** Opus (quality-critical; this is the one place we spend).

**Token win:** the writing churn (drafts, revisions, formatting) happens in an isolated context. Main receives the final file path, not the draft history.

### 2.3 Judge subagent — "Verifier"

**Input:** artifact path + rubric (judge.md, unchanged).

**Output:** `judges/judge-{id}-loop{n}.md` with existing format.

**Model:** Sonnet.

**Token win:** judge's re-reads of the artifact stay in its subagent context.

### 2.4 Refine — targeted diffs, not rewrites

Instead of re-running writer, main reads the judge's MUST list and either:
- **Small diff (≤3 MUSTs, all additive):** main itself does targeted `Edit` calls. Cheapest.
- **Large diff (≥4 MUSTs or structural):** spawn writer sub-agent in "refine mode" — prompt includes current artifact + MUST list + instruction to minimize unchanged sections.

---

## 3. Additive judge improvements (no existing rule removed)

All 7 criteria, scores, and PASS thresholds stay **exactly as written in v1 judge.md**. Additions:

### 3.1 Pre-write judge (new, optional)

Before writer spawns, a tiny judge pass scores the **outline** (a 10-line structural sketch produced by main from the brief) against `must_include`. Does NOT use the 7-criteria rubric — that is reserved for the finished artifact. Instead it answers one question: "Does every item in `must_include` map to at least one outline line?" Output: either `PASS` or a list of missing items in the form `must_include[k] → add section covering {topic}`. One Sonnet call, bounded output, no overlap with the main judge's scoring criteria.

### 3.2 Cross-artifact coherence judge (new, runs once at end)

After all artifacts pass, one Sonnet pass reads all artifacts' section headers + "Keep" lists and flags:
- Contradictions between artifacts (e.g., 01 says X is deprecated, 05 recommends X)
- Unclaimed must-includes (topic was promised in 00 but not delivered anywhere)
- Heavy duplication (same 500-word concept in two artifacts)

Does NOT re-score. Only emits a `coherence-report.md` with fix suggestions. Human decides.

### 3.3 Early-exit rule (codified)

If Loop 1 judge ≥ 85 AND no MUSTs → skip Loop 2. v1 already allowed this de facto; v2 codifies it so the orchestrator doesn't default to 2 loops.

### 3.4 Slop/Concision sub-check (additive to existing Slop/10)

Existing Slop criterion stays /10. Add a diagnostic note (not a new score): judge emits `concision_flag: true` if artifact exceeds min_words by >40% without added signal. Main uses this to decide targeted trims. No rubric change.

---

## 4. Parallelism policy

- **Artifacts 00 and 01 remain serial.** They set voice + foundation. 01 reads 00 as exemplar.
- **Artifacts 02–N can spawn in parallel batches of 2–3.** Each writer gets 00+01 as exemplars to hold voice. Each still runs its own judge loop.
- **Cap parallel writers at 3.** More risks rate limits + drift. User-configurable.
- **Risk mitigation:** exemplar injection + the cross-artifact coherence judge in §3.2 catches drift at the end.

---

## 5. Esoteric-bug ralph loop — critique

User's wishful thought: "leverage ralph loop to find esoteric bugs." Verdict: **out of scope, don't pursue in this skill.**

Ralph loop is a **refinement loop over known material** — it polishes study artifacts, it doesn't generate novel attack hypotheses. To discover esoteric bugs you'd need:
- A hypothesis-generator agent (attack-surface permutation)
- A primitive-hunter (targeted recon on recent CVEs, GitHub commits, CodeQL queries)
- A tester (actually hitting live apps or recreated labs)

None of these fit "produce study material." Forcing it would make artifacts worse (speculation instead of verified facts). Recommendation: keep research-mastery focused on synthesis. Discovery belongs in a separate skill with different primitives — out of scope for v2, full stop.

---

## 6. "Research first, rough draft first" — assessment

User's instinct matches how human researchers work: **gather → outline → draft → polish.** v2 encodes this as:

1. **Gather** = Scout subagent (§2.1) produces brief.md
2. **Outline** = Pre-write judge on 10-line sketch (§3.1)
3. **Draft** = Writer subagent (§2.2)
4. **Polish** = Judge → targeted refine (§2.4)

This is a straight upgrade over v1's "writer does everything in one pass." It also happens to be the main token saver, because §1 and §2 are cheap and prevent expensive §3-4 rework.

---

## 7. What stays unchanged (non-negotiable)

- 7-criteria rubric in `judge.md` — same criteria, same points, same PASS threshold (≥75).
- Voice rules in `writer.md` — banned words, WHY-first, signals-first, explain-at-the-bar tone.
- 3-tier research escalation — just moves into scout.
- `research-plan.json` schema.
- Output structure (`artifacts/`, `checklists/`, `maps/`).
- Artifact 00 is longest.
- Case study as final artifact.

---

## 8. What's new in v2 files

| File | Status | Purpose |
|---|---|---|
| `SKILL.md` (v2 dir) | new | Same shape as v1 but with subagent delegation steps |
| `references/scout.md` | new | Scout subagent prompt template + brief.md schema |
| `references/orchestrator.md` | new | How main holds state, when to spawn what |
| `references/judge.md` | copied unchanged | 7-criteria rubric preserved |
| `references/writer.md` | copied + brief-consumption addendum | Tells writer to start from brief.md, not raw sources |
| `references/refine.md` | copied + targeted-diff section | Adds the "small diff → Edit, large diff → writer sub" policy |
| `references/coherence.md` | new | Cross-artifact coherence judge prompt |

---

## 9. Token savings — measurement plan, not fabricated numbers

No baseline was captured during the OAuth v1 run, so any specific percentage here would be invented. The honest answer: we expect meaningful savings, but the size is unknown until measured.

**Measurement instrument (must be in place before the A/B):**

1. Each agent spawn logs to `research/{topic}/token-log.jsonl`:
   ```
   {"artifact":"02","agent":"scout","model":"sonnet","in":~,"out":~,"ts":"..."}
   ```
2. Main logs its own pre/post-spawn token counts from `/context` or usage meta.
3. After the test artifact, `token-log.jsonl` is summed per agent and compared to a v1 number captured for the baseline artifact.

**Directional intuition (not a number, a direction):**
- Dense-source artifacts (heavy RFCs) should benefit most from scout — that's where v1 drowned.
- Thin-source artifacts (blog posts, case studies) may barely move; scout overhead could even cost slightly.
- Refine-heavy artifacts benefit from targeted `Edit` over writer re-spawn.
- Early-exit saves a full loop when loop 1 ≥85 — binary win when it triggers.

If v2 doesn't measurably cut tokens on a dense-source artifact, it's not worth keeping. That's the decision the A/B in §10 exists to make.

---

## 10. Rollout — controlled A/B, same artifact type

Comparing OAuth Artifact 02 (v2) against OAuth Artifact 01 (v1) would confound *workflow* with *topic* — techniques vs redirect-URI exploitation are different difficulty profiles. The A/B must hold artifact *type* constant and vary only the workflow.

**Test design:**

1. Pick a fresh topic the user hasn't researched yet (e.g., SSRF, SSTI, or XXE).
2. Produce the `00-foundations` artifact under v1 first (captures a v1 token baseline for the *same artifact type* — foundations — that v1 already excels at).
3. Produce a `00-foundations` artifact for a *different* fresh topic under v2.
4. Compare: judge score, loops-to-pass, token cost (from `token-log.jsonl`).

Optional tighter test: produce the same topic's foundations artifact under both v1 and v2. Higher signal, roughly 2x the time cost. User's call.

**Separately, continue OAuth under v1.** OAuth is halfway done on v1; don't disrupt it. v2 A/B happens on a new topic.

**Adopt v2 only if all three hold:**
- v2 judge score ≥ v1 score − 3 (no real quality loss)
- v2 token cost meaningfully lower (target: −25% or more; below that, complexity isn't worth it)
- Scout brief-gate didn't fail more than once per artifact (signals scout is reliable)

**Otherwise:** keep v1, archive v2 proposal, move on.

---

## 11. Open questions for user

1. Do you want me to set up a lightweight token log (per-agent spawn, approximate tokens) so the v2 A/B test has real numbers, not estimates?
2. Parallelism cap — 3 is my default; comfortable, or go higher/lower?
3. For the scout subagent model — Sonnet is my default. Want Haiku (cheaper, risk: misses nuance in RFCs) or keep Sonnet?
