---
name: research-hunt-v3
description: >
  Jason Haddix-style source hunter for security topics. Spawns N fresh Sonnet subagents in sequence, each given the current research_links.md and asked to find rarer sources than prior rounds. Returns a curated 15-25 link list with rarity ratings and novel-pattern notes. Trigger: "research v3 <topic>", "hunt v3 on <topic>", "deep hunt <topic>".
---

# Research Hunt — v3

Different goal than v1/v2. **Not building artifacts. Hunting *sources* — the rarer the better.**

## What v3 does

For a security topic, run N (default 20) sequential rounds. Each round = a fresh **Sonnet subagent** (Task tool, `general-purpose`) that:
1. Reads `research_links.md` (everything found so far)
2. Hunts via WebSearch + WebFetch
3. Appends new high-signal sources with rarity ratings
4. Returns a one-paragraph status to main

Main exits when:
- N rounds complete, OR
- ≥20 ★4-5 sources accumulated (whichever first)

Final output: numbered curated list + novel-pattern notes.

## How v3 differs from v1/v2

| | v1/v2 | v3 |
|---|---|---|
| Goal | Build mastery artifacts | Hunt rare sources |
| Output | 5-10 written artifacts | One ranked link list |
| Subagents | Scout/Writer/Judge | Hunter only |
| Sources | Means to an end | The end itself |
| Cost profile | High (Opus writers) | Low (Sonnet hunters) |
| Context model | Each subagent sticky | Each round fresh context |

## Step 1 — Setup (main)

1. Slugify topic. Create run dir: `research/runs/{topic-slug}-{YYYY-MM-DD}/`.
2. Initialize `research_links.md` with topic header + any user-provided seed URLs (each tagged `[seed]`, no rating).
3. Initialize `round-log.md` (one line per round summary).

## Step 2 — Round loop (main)

For round in 1..N:
- Spawn **Task subagent** with:
  - `subagent_type: general-purpose`
  - `model: sonnet`
  - `prompt: references/hunter-prompt.md` template, filled with: topic, round number, full content of `research_links.md`, and depth ladder for this round (see below)
- Subagent edits `research_links.md` directly (Write tool), returns short status.
- Append status line to `round-log.md`.
- Check termination: if ≥20 ★4-5 entries OR round == N, break.

## Step 3 — Curate (main)

Spawn final **curator subagent** (Sonnet, single call) with `references/curator-prompt.md`:
- Reads final `research_links.md`
- Sorts by rarity (★5 first, then ★4, then ★3 if needed)
- Selects top 15-25 entries
- Writes `final-curated.md` with:
  - Numbered list (title + URL + 1-line gold reason)
  - "Novel patterns spotted" section (cross-source synthesis: what techniques showed up that aren't in mainstream guides?)

Main reads `final-curated.md` and presents to user.

## Round depth ladder

| Rounds | Hunt mode | Example sources |
|---|---|---|
| 1-5 | Foundational | Top researchers' index pages, canonical writeups (PortSwigger, HackTricks, OWASP) |
| 6-12 | Mid-tier | Bug bounty disclosure writeups, GitHub PoC repos, less-cited blog posts, Medium deep dives |
| 13-20 | Deep cuts | Conference slides (PDF), archived forum threads, GitHub issues (not READMEs), gist comments, twitter threads, personal sites of niche researchers, bachelor's/master's theses |

The hunter prompt template embeds the depth instruction for the current round.

## Rarity rating

| Stars | Meaning | Discoverability |
|---|---|---|
| ★5 | Singular insight, mentioned 1-5 times on internet | Found via creative search operators only |
| ★4 | Niche but discoverable | <100 incoming refs, 2nd or 3rd page of results |
| ★3 | Solid mid-tier | Top-50 results |
| ★2 | Top-10 Google | Adds value but well-known |
| ★1 | Generic | Drop unless seed |

Hunter must justify ★4-5 rating in the entry (e.g., "Google returns 7 results for this URL pattern" or "linked from only 2 other security blogs").

## State file: research_links.md

```
# Research: {topic}

## Seeds
- [Title](URL) — provided by user [seed]

## Round 1
- ★3 [Title](URL) — 1-sentence why this is gold.
- ★4 [Title](URL) — Justification: <discoverability note>.

## Round 2
...
```

**Append-only.** Hunter MUST NOT remove or rewrite prior entries.

## Termination

- Hard stop: N rounds (default 20).
- Soft stop: ≥20 entries with ★4 or ★5.
- Both met early: stop.

## Anti-hallucination rules

- Hunter MUST WebFetch every URL it adds. Title and 1-line insight come from real fetched content, not invented.
- If WebFetch returns 404 / paywall / no content, link is dropped (not added).
- If hunter cannot find any new sources in 3 search attempts, it returns `"exhausted: <reason>"` and main moves on (counts as a round).
- No re-adding of links already in `research_links.md`. Hunter checks first.

## Model assignments

| Role | Model | Why |
|---|---|---|
| Main orchestrator | Opus (current session) | Planning + sequencing + final presentation |
| Hunter subagent (×N) | Sonnet 4.6 | Search + judgment, ~$0.05-0.15 per round |
| Curator subagent (×1) | Sonnet 4.6 | Cross-source synthesis at end |

**Cost target on $20 plan:** ~$1.50-4 per full 20-round run.

## Reference files

| File | Read when |
|---|---|
| `references/hunter-prompt.md` | Spawning each round's hunter |
| `references/curator-prompt.md` | Final curation step |

## Invocation example

User: *"research v3 advance BAC methods and IDOR hunting"*

Main:
1. Creates `research/runs/bac-idor-2026-04-25/research_links.md` with seeds.
2. Loops 20 rounds, each a fresh Sonnet hunter.
3. Curator pass.
4. Outputs final list inline.
