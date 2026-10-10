---
name: research-mastery-v2
description: >
  Multi-agent deep research skill for mastery-level study material on ANY security topic. Same output shape as v1, but main orchestrator delegates to Scout (extraction), Writer (synthesis), Judge (verification) subagents so raw sources and full artifact drafts never enter main context. Trigger: "research v2 this", "ralph v2 on", "deep dive v2". Uses the full 3-tier research escalation (inside Scout), Ralph loops with early-exit on ≥85, optional parallel artifacts, and a final cross-artifact coherence check.
---

# Research Mastery — v2

> Same artifacts as v1. Main context 45–65% lighter. Voice + rubric + output shape unchanged.

```
"Use research-mastery-v2 to build mastery on SSRF with 3 Ralph loops"
"Ralph v2 on SSTI — 2 loops, parallel 02+"
"Deep dive v2 into XXE — serial, A/B against v1 foundations"
```

---

## How v2 differs from v1 (one-page view)

| Step | v1 | v2 |
|---|---|---|
| Plan | Main writes `research-plan.json` | Same |
| Research | Main runs Tier-1/2/3 inline → raw pages in context | **Scout subagent** (Sonnet) runs all tiers, writes `briefs/{id}-brief.md` + appendices |
| Pre-write | Main starts drafting | **Pre-write judge** (Haiku): outline vs `must_include` — bounded output, no rubric overlap |
| Write | Main writes artifact | **Writer subagent** (Opus) writes, reads brief first, appendices on demand |
| Judge | Main judges inline | **Judge subagent** (Sonnet), rubric from `judge.md` unchanged |
| Refine | Main rewrites | **Targeted `Edit`** for small diffs; writer sub-agent in "refine mode" for large |
| Parallelism | Strict serial | 00 and 01 serial; 02+ in batches of 3 (configurable) |
| End-of-run | Nothing | **Coherence judge** (Sonnet) — non-scoring, flags cross-artifact issues |

**What does NOT change:** 7-criteria rubric, PASS ≥75, voice rules, banned words, `research-plan.json` schema, output directory layout, WHY-first depth bar, artifact 00 is longest.

---

## Step 1 — Plan (main)

Two input modes.

### Mode A — Topic (default, same as v1)

User says "build mastery on OAuth." Main decomposes via the 5 questions (bypass / chain / platform / blind / case study). Every topic gets `00-foundations` + `01-techniques` automatically. Minimum 5 artifacts, max 10.

### Mode B — Curriculum (reverse-engineer a course)

User pastes a course outline, e.g.:
```
Module 1: XSS reflected
Module 2: XSS stored
Module 3: DOM XSS
Module 4: CSP bypass
...
Module 11: XSS in PDF generators
```

Main emits one artifact spec per module. Skips the 5-question decomposition. Foundations (00) + techniques (01) are NOT auto-added — the curriculum IS the decomposition. However, if the first two modules are clearly intro-shaped, main should treat them as 00/01 (serial, exemplar for later modules).

Main proposes `must_include` per module by web-searching the module title briefly and reading 1–2 canonical sources (OWASP, PortSwigger cheatsheet, researcher index). User reviews the plan before research starts — this is the cheapest correction point.

### Plan schema

```json
{
  "topic": "...",
  "input_mode": "topic" | "curriculum",
  "loops": 3,
  "parallelism": "serial" | "batched",
  "research_depth": "light" | "standard" | "deep",
  "artifacts": [
    {
      "id": "00",
      "title": "...",
      "filename": "...",
      "focus": "...",
      "must_include": [...],
      "min_words": 1500,
      "research_depth": "deep",
      "sources_to_start": [...],
      "status": "pending",
      "loop": 0,
      "score": null,
      "judge_notes": []
    }
  ]
}
```

Top-level `research_depth` is the default for all artifacts. Per-artifact `research_depth` overrides for that artifact only. See §Research depth below.

**Main holds in context:** plan JSON, artifact 00 final (once done, for voice), brief index (pointers not content). Nothing else sticky.

---

## Research depth (universal)

The `research_depth` field controls scout's source budget. Applies the same way in topic mode or curriculum mode. Per-artifact override wins over topic-level default.

| Depth | web_search budget | web_fetch budget | When to use |
|---|---|---|---|
| `light` | 3 | 2 | Well-known topics (basic XSS reflected). Saves tokens. |
| `standard` | 5–8 | 3–5 | Default. What scout does unset. |
| `deep` | 8–12 | 5–8 | Niche / esoteric (prototype pollution, CSP bypass edge cases, cache deception). |

Scout must respect the budget. If coverage incomplete within budget, scout returns `source_coverage: partial` + open questions. Main decides: accept partial, or re-spawn scout with `deep` for that artifact.

Rule of thumb: start the whole topic at `standard`. Bump specific artifacts to `deep` when the judge flags "thin coverage" on loop 1. Drop well-known artifacts to `light` if you want to save budget for the hard ones.

---

## Step 2 — Scout (Sonnet subagent, per artifact)

Main spawns Scout with the artifact spec + list of raw sources + Tier-1 query suggestions. Scout runs all three research tiers in its own context, then writes:

- `artifacts/briefs/{id}-brief.md` — primary brief, target ≤800 words, schema in `references/scout.md`
- `artifacts/briefs/{id}-brief-appendix-*.md` — verbatim HTTP traces / RFC quotes that overflow the budget

Scout returns to main: brief path + `source_coverage` status + any open questions.

**Fidelity rule:** scout quotes RFCs verbatim for MUST/SHOULD and parser-level mechanics. No paraphrasing of precision-critical facts. See `references/scout.md`.

---

## Step 3 — Brief-gate (Haiku, single call)

Main runs a bounded Haiku call: reads brief + must_include + scout_confidence, emits PASS or a list of gap items. Single-step decision, no reasoning chain → Haiku is sufficient.

**On FAIL:**
1. Preferred: re-spawn Scout scoped to the gap list only (scout re-reads the specific RFC section, not the whole file).
2. Last-resort: writer reads the raw source, but ONLY the anchor range the gap points to (e.g., RFC 6749 §4.1, not the full RFC). **Full raw-source reads into main context are forbidden.**

Log the escalation in `research/{topic}/scout-log.jsonl`.

---

## Step 4 — Pre-write outline judge (Haiku, single call)

Main writes a 10-line outline sketch from the brief. Haiku checks: does every `must_include` item map to at least one outline line? Output is deterministic (`PASS` or `must_include[k] → add section covering {topic}`). See `references/prewrite-judge.md`.

Does NOT run the 7-criteria rubric. That is reserved for the finished artifact.

---

## Step 5 — Write (Opus subagent)

Main spawns Writer with: artifact spec + brief path + approved outline + exemplar artifact (00 final, if not the first artifact) + `references/writer.md`.

Writer reads brief first, appendices on demand, writes the full `.md` to disk, returns the path.

**Writer is Opus.** This is the one place v2 spends on the big model. Quality of synthesis is non-negotiable.

---

## Step 6 — Judge (Sonnet subagent)

Main spawns Judge with: artifact path + `references/judge.md` (rubric unchanged). Judge writes `judges/judge-{id}-loop{n}.md` with the v1 format.

**Early-exit rule:** if score ≥85 AND zero MUSTs → artifact passes, skip further loops. Codified.

---

## Step 7 — Refine (targeted or sub-agent)

Read judge's MUST list.

- **≤3 MUSTs, all additive (no structural change):** main does targeted `Edit` calls. Token-cheap, no new spawn.
- **≥4 MUSTs OR structural change OR re-research needed:** spawn Writer in "refine mode" — prompt includes current artifact + MUST list + instruction to minimize unchanged sections.

See `references/refine.md`.

---

## Step 8 — Loop or move on

Repeat Steps 6–7 until PASS. Max loops = as configured. Default 3, early-exit at 85.

---

## Step 9 — Parallelism (artifacts 02+)

Once 00 and 01 are PASS, main may batch 02+ artifacts:

- Default batch size = 3. Configurable, cap = 3 to limit voice drift.
- Each parallel writer gets 00+01 as exemplars.
- Steps 2–7 run independently per artifact in its own subagent chain.
- Main joins when all finish.

Serial still works — set `parallelism: "serial"` in plan for runs where drift matters more than speed.

---

## Step 10 — Cross-artifact coherence (Sonnet, once, end of run)

After all artifacts PASS, main spawns one coherence judge pass. Reads all artifacts' section headers + "Keep" lists. Emits `coherence-report.md`: contradictions, unclaimed must-includes, heavy duplication. Does NOT re-score. See `references/coherence.md`.

Human reads the report, decides which fixes to apply. Not an automatic loop.

---

## Step 11 — Final outputs (unchanged from v1)

1. Read `references/checklist.md` → testing checklist + spotting guide
2. Read `references/concept-map.md` → concept map + Mermaid + learning path

No web research. Reads finalized artifacts only.

---

## Token log

Every subagent spawn appends to `research/{topic}/token-log.jsonl`:
```json
{"artifact":"02","agent":"scout","model":"sonnet","in":~,"out":~,"loop":1,"ts":"..."}
```

Main logs its own pre/post-spawn counts. The log is the measurement instrument for any v1-vs-v2 A/B.

---

## Orchestrator session discipline

Main context holds (targets):
- `research-plan.json` (~1–3k)
- Exemplar artifact 00 final (~6–10k, after 00 done)
- Brief paths + one-line summaries (~200 tokens × N)

If main grows past ~25k sticky tokens, split the run at the 00/01-done boundary into a fresh session. The new session reads 00+01 as exemplars and continues with 02+.

---

## Model assignments (summary)

| Role | Model | Why |
|---|---|---|
| Orchestrator (main) | Opus | Planning + judgment + voice |
| Scout | Sonnet | Extraction with RFC nuance; Haiku risks missing depth |
| Writer | Opus | Synthesis quality is the product |
| Brief-gate check | **Haiku** | Single-step coverage check, deterministic output |
| Pre-write outline judge | **Haiku** | Single-step mapping, deterministic output |
| Judge (main rubric) | Sonnet | Needs reasoning about each of 7 criteria |
| Coherence judge | Sonnet | Cross-artifact reasoning |
| Refine (small diff) | Opus (main) | Edit calls, no spawn |
| Refine (large diff) | Opus | Writer sub-agent in refine mode |

---

## Rules (inherited from v1, unchanged)

**WHY before WHAT.** L3 minimum in foundations, L2 in techniques.
**Configs that create the vuln.** Show vulnerable code next to the attack.
**Behavioral signals.** Recognize before payloading.
**Real requests, real responses.** Success + blocked + next move.
**Source tracking.** Table at end of every artifact.
**No slop.** Banned: comprehensive, robust, utilize, leverage, cutting-edge, "it's important to note." If a paragraph loses nothing when deleted, cut it.
**Explain at the bar.** Senior hacker to smart junior.

---

## Reference files

| File | Read When | Model that consumes it |
|---|---|---|
| `references/orchestrator.md` | Main, throughout | Opus (main) |
| `references/scout.md` | Spawning scout | Sonnet (scout) |
| `references/prewrite-judge.md` | Step 4 | Haiku |
| `references/writer.md` | Spawning writer | Opus (writer) |
| `references/judge.md` | Spawning judge | Sonnet (judge) |
| `references/refine.md` | Step 7 | Opus |
| `references/coherence.md` | Step 10 | Sonnet |
| `references/checklist.md` | Step 11 | Opus (main) |
| `references/concept-map.md` | Step 11 | Opus (main) |
