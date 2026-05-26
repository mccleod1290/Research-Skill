# Research Agent Operating Guide

This workspace contains three research workflows:

- `research skill/` is the quality bar: deep artifacts, WHY-first writing, Ralph judge/refine loops, source tables, checklists, and concept maps.
- `research skill v2/` is the token-saving architecture: orchestrator, scout briefs, writer, judge, targeted refinement, early exits, and token logs.
- `research skill v3/` is the low-cost source hunter: repeated cheap hunter rounds that find rare, high-signal sources before writing begins.

Use this file as the default agent instruction set for new research runs. The goal is not to run all three workflows at full blast. The goal is to borrow the best part of each one without wasting context.

## Default Strategy: Hybrid Research Loop

Use the Hybrid Research Loop unless the user explicitly asks for v1, v2, or v3 alone.

1. **Plan from v1.** Decompose the topic into artifacts with `00-foundations` and `01-techniques` unless the user provides a curriculum. Keep artifact count small by default: 4-7 artifacts, max 10 only for deep runs.
2. **Hunt from v3.** Run a short source hunt before scouting. Default to 3-5 hunter rounds, not 20. Use 8-12 rounds only for niche topics or when the user asks for rare sources.
3. **Orchestrate from v2.** Main context coordinates only. It keeps the plan, file paths, short summaries, and final exemplars. Raw sources, long briefs, drafts, and judge details live on disk or inside subagents.
4. **Write to the v1 bar.** Artifacts must explain WHY, show vulnerable configs/code, include behavioral signals, real requests/responses, source tables, and concise practitioner prose.
5. **Refine cheaply.** Judge cold, fix targeted gaps, and early-exit when quality is high. Do not rewrite whole artifacts to fix small issues.

## Token Rules

- Prefer file paths over pasted content.
- Prefer search snippets over full page fetches when the snippet answers the question.
- Fetch only high-signal sources. Skip SEO pages, generic introductions, and posts without real HTTP/code examples.
- Never load raw RFCs, long blog posts, or whole artifact drafts into main context.
- Scout writes `briefs/{id}-brief.md` at about 800 words. Overflow goes into appendices.
- Writer reads the brief first and appendices only when a section needs exact HTTP traces, RFC language, or canonical examples.
- Judge reports return scores, MUST fixes, SHOULD fixes, and keep-list. Main reads only what is needed for the next edit.
- Use targeted edits for three or fewer additive MUST fixes. Spawn/refocus a writer only for structural changes, many MUSTs, or missing research.
- Split the run after artifacts `00` and `01` if the session gets heavy. A fresh session can resume from `research-plan.json` plus `00`/`01` exemplars.

## Cost Modes

Choose the cheapest mode that can satisfy the user.

| Mode | Use When | Hunt | Research Depth | Loops | Parallelism |
|---|---|---:|---|---:|---|
| `lean` | User wants affordable, fast, or broad coverage | 1-3 rounds | light/standard | 1-2 | serial or batch 2 |
| `standard` | Default for new topics | 3-5 rounds | standard | 2, early-exit at 85 | 00/01 serial, then batch 2 |
| `deep` | Niche, high-stakes, or user asks for mastery | 8-12 rounds | standard/deep per artifact | 3, early-exit at 85 | batch up to 3 |
| `source-hunt-only` | User asks for links, rare sources, or reading list | 10-20 rounds | none | none | hunter sequence |

Default to `standard`. Use `lean` when the user mentions budget, low token use, speed, or the $20 plan. Use `deep` only when the user asks for depth or the judge proves standard coverage is thin.

## Agent Roles

### Orchestrator

Main agent. Plans, routes, updates status, and keeps context small.

Holds:
- `research-plan.json`
- final `00` and optionally final `01` as voice exemplars
- brief paths plus one-line summaries
- judge report paths plus score/MUST summary
- token log path

Does not hold:
- raw sources
- full source dumps
- full draft history
- line-by-line judge reasoning
- all artifacts at once

### Hunter

Borrowed from v3. Finds sources before artifact scouting.

Default behavior:
- Create `research/runs/{topic-slug}-{YYYY-MM-DD}/research_links.md`.
- Run 3-5 rounds for standard mode.
- Each round must fetch every added URL.
- Rate sources with stars.
- Append only. Never rewrite prior rounds.
- Stop early if enough high-signal sources exist for the plan.

Use full v3 only when the user asks for source hunting or rare/deep cuts.

### Scout

Borrowed from v2. Extracts, does not synthesize.

Input:
- one artifact spec
- source seeds from `research_links.md`
- artifact research depth
- suggested queries

Output:
- `artifacts/briefs/{id}-brief.md`
- optional appendix files
- `source_coverage: complete | partial`
- open questions

Rules:
- Quote RFC/vendor/CVE/parser language verbatim where precision matters.
- Cite every fact with URL and section/anchor when available.
- If a must-include item is weak, mark it `partial` or `low`; do not hide the gap.
- Do not exceed the source budget unless the orchestrator explicitly bumps the artifact to `deep`.

### Writer

Borrowed from v1/v2. Writes the artifact from the scout brief.

Rules:
- Read brief, outline, and exemplar first.
- Read appendices only on demand.
- Use raw source only for a named anchor range that the scout flagged.
- Foundations require deep WHY chains. Techniques require at least clear HOW/WHY.
- Every practical technique needs request, success response, blocked/failure response, and next move.
- End every artifact with a source table.
- Avoid filler and banned phrases from `research skill*/references/writer.md`.

### Judge

Borrowed from v1/v2. Reviews cold.

Use the 100-point rubric:
- Foundation Depth /25
- Behavioral Signals /15
- Request/Response /20
- Practitioner Ready /15
- Anti-Slop /10
- Accuracy /10
- Gaps /5

Pass rules:
- `>=85` and no MUST fixes: pass and early-exit.
- `75-84` and no MUST fixes: pass; log SHOULD fixes as optional.
- `<75` or any MUST fixes: revise.
- `<50`: rewrite or rebuild the brief.

### Curator / Coherence Judge

Borrowed from v2/v3. Runs at the end.

For full artifact runs:
- Read artifact headings, keep-lists, source tables, and summaries.
- Flag contradictions, duplicated sections, and promised-but-missing coverage.
- Do not rescore unless the user asks.

For source-hunt-only runs:
- Curate top 15-25 links from `research_links.md`.
- Prefer rare, actionable, diverse sources.
- Include novel patterns only when supported by multiple high-rated links.

## Planning Rules

For a topic prompt, ask five decomposition questions internally:

1. Are there distinct bypass categories?
2. Are there protocol or chain variants?
3. Are there platform/framework-specific variants?
4. Is there a blind or out-of-band variant?
5. Is there a strong real-world case study?

Always include `00-foundations` and `01-techniques` for topic mode.

For a curriculum prompt, treat modules as the decomposition. Do not auto-add foundations/techniques unless the curriculum clearly starts with them.

Keep the plan reviewable:
- artifact id
- title
- filename
- focus
- must_include
- research_depth
- sources_to_start
- min_words
- status
- score
- loop

## Research Depth

| Depth | Searches | Fetches | Use For |
|---|---:|---:|---|
| `light` | 3 | 2 | common/basic artifacts |
| `standard` | 5-8 | 3-5 | default |
| `deep` | 8-12 | 5-8 | niche/esoteric artifacts or judge-flagged thin coverage |

Start standard. Upgrade only the artifact that needs it.

## Output Layout

Use this layout for hybrid runs:

```text
research/runs/{topic-slug}-{YYYY-MM-DD}/
├── research-plan.json
├── research_links.md
├── round-log.md
├── token-log.jsonl
├── artifacts/
│   ├── 00-foundations.md
│   ├── 01-techniques.md
│   ├── ...
│   └── briefs/
│       ├── 00-brief.md
│       └── 00-brief-appendix-1.md
├── judges/
│   └── judge-00-loop1.md
├── checklists/
│   ├── testing-checklist.md
│   └── spotting-guide.md
├── maps/
│   ├── concept-map.md
│   └── concept-map.mermaid
├── coherence-report.md
└── final-curated.md
```

## Token Log

Append a line after each meaningful step when usage data is available. If exact token counts are unavailable, write `null` and still log the event.

```json
{"artifact":"02","agent":"scout","model":"cheap-capable","mode":"standard","loop":1,"in":null,"out":null,"ts":"2026-05-26T00:00:00+05:30"}
```

Use the log to decide whether the hybrid workflow is actually saving cost. If token logging is missing, do not claim savings as fact.

## When To Ask The User

Make reasonable assumptions by default. Ask a short follow-up only when:

- the target output is unclear: source list, study artifacts, checklist, concept map, or all of them
- the budget mode matters and the user did not imply one
- the topic is too broad for a useful plan
- the user wants offensive/security content but authorization or scope is unclear
- a repeated judge/scout failure suggests the plan itself is wrong

If the user says "affordable", "token optimized", "low cost", or similar, choose `lean` or `standard` automatically instead of asking.

## Invocation Examples

```text
Use the hybrid research loop on SSRF, standard mode, 2 loops.
```

```text
Use lean mode to build a study pack on IDOR/BAC. Keep it affordable.
```

```text
Run source-hunt-only on advanced access control bypasses, 12 rounds, then curate.
```

```text
Resume research/runs/access-control-gadgets-v3-2026-05-26 from artifact 02.
```

## Non-Negotiables

- Quality comes from v1: WHY before WHAT, real examples, no filler.
- Savings come from v2: briefs, gates, targeted edits, early exits, context hygiene.
- Discovery comes from v3: cheap hunter rounds before expensive writing.
- Do not spend Opus-level effort on link hunting, source extraction, or checklist gates if a cheaper capable model/tool is available.
- Do not run 20 hunter rounds, 10 artifacts, and 3 loops unless the user explicitly asks for a deep run.
- Do not claim a source says something unless it was fetched or appears in a trusted local file with a citation.
