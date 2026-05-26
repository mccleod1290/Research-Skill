# Research Skill

This repo is a security research workflow kit for building practitioner-grade study packs from a topic such as SSRF, SSTI, XXE, IDOR/BAC, OAuth, cloud metadata abuse, parser differentials, GraphQL authorization, WebSocket authz, or mobile API authorization.

It is designed for authorized cybersecurity learning and research: turn broad topics into grounded artifacts with root-cause explanations, vulnerable configuration/code patterns, HTTP request/response examples, behavioral signals, source tables, checklists, and concept maps.

## Repository Map

| Path | Purpose |
|---|---|
| `AGENTS.md` | Default coordinator. It combines the best parts of v1, v2, and v3 into one hybrid research loop. |
| `research skill/` | v1 quality bar. Single-agent deep research, WHY-first writing, Ralph judge/refine loops, checklists, and concept maps. |
| `research skill v2/` | Token-aware multi-agent architecture. Orchestrator, scout briefs, writer, judge, targeted refinement, parallelism, and token logs. |
| `research skill v3/` | Low-cost source hunter. Repeated cheap hunter rounds to find rare, high-signal sources before writing begins. |
| `research/runs/` | Generated research runs, source lists, briefs, judge reports, final curated outputs, checklists, and maps. |

## Cybersecurity Use Cases

- Build mastery packs for a vulnerability class: foundations, techniques, bypasses, platform variants, blind/OOB variants, and real-world case studies.
- Convert a course outline into open-source study artifacts, one module at a time.
- Prepare bug bounty or internal testing methodology with behavioral signals, target inputs, safe test flow, blocked responses, and next moves.
- Investigate complex chains such as URL parser differential -> SSRF, cloud metadata token exposure, GraphQL IDOR, WebSocket authz gaps, or OAuth redirect/PKCE mistakes.
- Hunt rare sources before writing: obscure writeups, RFC sections, CVE notes, conference slides, GitHub issues, implementation bugs, and researcher posts.
- Produce final learning aids: testing checklists, spotting guides, concept maps, Mermaid diagrams, and curated source lists.

## How Complex Research Runs Work

The default workflow is the Hybrid Research Loop from `AGENTS.md`:

1. Plan from v1: decompose the topic into a small set of artifacts, usually `00-foundations`, `01-techniques`, then 2-5 focused variants.
2. Hunt from v3: run short source-hunt rounds first, so the writers start with better sources instead of generic SEO pages.
3. Orchestrate from v2: keep the main context small. Store raw sources, scout briefs, drafts, and judge reports on disk.
4. Write to the v1 bar: every artifact explains why the issue exists, what code/config creates it, what behavior reveals it, and what HTTP evidence confirms it.
5. Refine cheaply: judge cold, fix targeted MUST gaps, and stop early when the artifact passes quality gates.

The output layout is intentionally file-based:

```text
research/runs/{topic-slug}-{YYYY-MM-DD}/
+-- research-plan.json
+-- research_links.md
+-- round-log.md
+-- token-log.jsonl
+-- artifacts/
|   +-- 00-foundations.md
|   +-- 01-techniques.md
|   +-- briefs/
+-- judges/
+-- checklists/
+-- maps/
+-- coherence-report.md
+-- final-curated.md
```

## Version Tradeoffs

| Version | Best At | Where It Lacks |
|---|---|---|
| v1: `research skill/` | Highest writing standard, deep WHY chains, tight narrative, strong final artifacts, simple mental model. | Expensive in context/tokens because one session reads sources, writes, judges, and refines everything. Slower for large topics. |
| v2: `research skill v2/` | Large research runs with lower context pressure. Scouts extract, writers synthesize, judges score, and the orchestrator mostly passes file paths. Good for 5-10 artifacts. | More moving parts. Quality depends on good briefs and clean handoffs. Rare sources still need good seeds or a v3 hunt first. |
| v3: `research skill v3/` | Cheap source discovery. Good for rare links, deep cuts, bug bounty writeups, PDFs, GitHub issues, and non-obvious research angles. | Does not write mastery artifacts by itself. It needs v1/v2 to turn sources into polished study material. |
| Hybrid via `AGENTS.md` | Best default: v1 quality, v2 orchestration, v3 discovery, with cost modes and early exits. | Requires discipline: keep raw sources out of main context, log runs, and avoid over-hunting common topics. |

## How `AGENTS.md` Coordinates Everything

`AGENTS.md` is the operating guide for new research runs. It chooses the default mode, roles, cost profile, output paths, and quality gates.

- Uses `standard` mode by default: 3-5 hunter rounds, standard research depth, 2 judge/refine loops, early exit at score 85.
- Switches to `lean` when speed or budget matters, `deep` when the topic is niche or judge coverage is thin, and `source-hunt-only` when the user only wants links.
- Defines agent roles: Orchestrator, Hunter, Scout, Writer, Judge, and Curator/Coherence Judge.
- Protects context: main keeps plans, file paths, summaries, scores, and exemplars; raw sources and long drafts live on disk or inside subagents.
- Enforces quality: WHY before WHAT, vulnerable code/config, real requests/responses, behavioral signals, source tables, and targeted fixes instead of full rewrites.

## Quick Prompts

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

## Practical Rule

Use v3 when you do not yet trust the source pool, v2 when the run is large enough to need orchestration, and v1 as the quality standard every artifact must satisfy.
