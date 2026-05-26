# Research Mastery — README

> You hand in a topic. You get back a folder of study documents that read like a senior practitioner wrote them. This README explains how the machine works, how to drive it, and how to stop it eating your tokens.

---

## Table of contents

1. [What this skill actually does](#1-what-this-skill-actually-does)
2. [v1 vs v2 — which one to use](#2-v1-vs-v2--which-one-to-use)
3. [The one-paragraph mental model](#3-the-one-paragraph-mental-model)
4. [The multi-agent picture (no jargon)](#4-the-multi-agent-picture-no-jargon)
5. [How to run it — step by step](#5-how-to-run-it--step-by-step)
6. [What you get out](#6-what-you-get-out)
7. [Token & context optimization — practical tips](#7-token--context-optimization--practical-tips)
8. [The $20 plan survival guide](#8-the-20-plan-survival-guide)
9. [Common failure modes & how to spot them](#9-common-failure-modes--how-to-spot-them)
10. [Glossary](#10-glossary)

---

## 1. What this skill actually does

Imagine you hire a study-buddy researcher. You say "I want to master OAuth." They:

1. **Plan** — decide the right number of documents to produce (foundations, techniques, case study, etc).
2. **Research** — for each document, search the web, read RFCs, read blog posts, extract the important parts.
3. **Write** — turn that into a clean study document written like a senior practitioner explaining it to you.
4. **Review** — re-read what they wrote, catch missing pieces or fluff.
5. **Revise** — fix what they caught, until the document is solid.
6. **Polish** — at the end, check all the documents agree with each other and aren't repetitive.

That's the whole skill. The trick is that **step 2 reads a LOT** (big RFCs, blog posts, everything), and if the same Claude session does all 6 steps, its memory (context window) fills up with raw research and it gets slow + expensive.

**v2's move:** split the work across multiple Claude agents so the main one never holds the raw research. More on that below.

---

## 2. v1 vs v2 — which one to use

| Situation | Use |
|---|---|
| Your current topic is halfway done on v1 | Keep using v1 |
| You're starting fresh | **v2** |
| You hit the 5-hour Opus limit a lot | **v2** |
| You want the absolute simplest workflow | v1 |
| You want maximum quality and don't care about cost | v1 (all-Opus) |
| You care about quality AND cost | **v2** |

**Don't mix them on the same topic.** Finish a topic on whichever version you started.

Both produce the same kind of output. v2 is faster and cheaper because it splits the work across smaller, specialized agents.

---

## 3. The one-paragraph mental model

**v1** is one Claude doing everything in one long session. By artifact 5 the session has read 6 RFCs + 20 blog posts + written 4 artifacts and judged them all. Huge context, slow, eats the 5-hour Opus limit.

**v2** is a small team. The main Claude is a project manager who never reads raw material. It spawns a Scout (cheaper Claude, Sonnet) that reads the raw stuff and hands back a short brief. It spawns a Writer (expensive Claude, Opus) that writes the artifact from the brief. It spawns a Judge (Sonnet) that scores the artifact. The main Claude just holds the plan and passes file paths around. Most of the expensive reading happens in throwaway sub-sessions whose memory is discarded when they finish.

---

## 4. The multi-agent picture (no jargon)

### The team

| Role | Who plays it | Job | Analogy |
|---|---|---|---|
| **Orchestrator** | Main Claude (Opus) | Plans, passes file paths, decides what happens next | Project manager |
| **Scout** | Sub-Claude (Sonnet) | Reads RFCs/blogs, writes a short brief | Research assistant |
| **Writer** | Sub-Claude (Opus) | Turns the brief into a full study document | Technical writer |
| **Judge** | Sub-Claude (Sonnet) | Scores the document against a rubric | Editor |
| **Brief-gate** | Sub-Claude (Haiku) | Quick check: did the scout cover everything? | Checklist |
| **Pre-write gate** | Sub-Claude (Haiku) | Quick check: does the outline cover everything? | Checklist |
| **Coherence judge** | Sub-Claude (Sonnet) | At end of run, checks all documents agree | Chief editor |

**Why different Claudes for different jobs?** Claude comes in three sizes:
- **Opus** — smartest, most expensive, slowest. Use for writing and planning.
- **Sonnet** — middle. Use for extraction and judgment.
- **Haiku** — cheapest, fastest, dumbest. Use for simple yes/no checklists.

v2 uses each one where it fits. Opus does the thinking-heavy work. Sonnet does the pattern-matching work. Haiku does the "did you include X" checks.

### The flow for one document

```
 ┌────────────┐
 │  Main      │ — has the plan
 └──┬─────────┘
    │ "Scout, read RFC 6749 §4.1, give me a brief"
    ▼
 ┌────────────┐
 │  Scout     │ — reads RFC, writes brief.md
 └──┬─────────┘
    │ returns brief path
    ▼
 ┌────────────┐
 │  Main      │ — "Haiku, does this brief cover all 7 must-haves?"
 └──┬─────────┘
    │ PASS / FAIL
    ▼
 ┌────────────┐
 │  Main      │ — writes 10-line outline, "Haiku, does this outline cover all 7?"
 └──┬─────────┘
    │ PASS / FAIL
    ▼
 ┌────────────┐
 │  Writer    │ — reads brief + outline, writes artifact.md
 └──┬─────────┘
    │ returns artifact path
    ▼
 ┌────────────┐
 │  Judge     │ — reads artifact, writes judge-report.md with score
 └──┬─────────┘
    │ score + MUSTs
    ▼
 ┌────────────┐
 │  Main      │ — score ≥85 no MUSTs? ship. else refine and re-judge.
 └────────────┘
```

The main Claude's memory only ever holds paths (`briefs/02-brief.md`, `judges/judge-02-loop1.md`) and a one-line summary. Never the full text.

---

## 5. How to run it — step by step

### The minimum prompt

```
Use research-mastery-v2 to build mastery on SSRF with 3 Ralph loops
```

Breakdown:
- `research-mastery-v2` — the skill
- `build mastery on SSRF` — your topic
- `with 3 Ralph loops` — how many judge-refine cycles per document (default 3)

### Options you can add

- `— 2 loops` → fewer refinement cycles (saves tokens, risks quality)
- `— serial` → run documents one at a time (v2 parallelizes documents 02+ by default)
- `— batch 2` → run 2 documents in parallel instead of 3
- `— no case study` → skip the final real-world case study document
- `— depth light` / `— depth standard` / `— depth deep` → research budget per document (default standard)

### Two input modes

**Mode A — Topic (default):** you name a topic, skill decomposes it into ~5–10 documents.
```
Use research-mastery-v2 to build mastery on SSRF
```

**Mode B — Curriculum (reverse-engineer a course):** you paste a course outline, skill produces one document per module, researched from open-source material.
```
Use research-mastery-v2 on this curriculum, depth standard:

Module 1: XSS reflected
Module 2: XSS stored
Module 3: DOM XSS
Module 4: CSP bypass basics
Module 5: CSP bypass — dangling markup
Module 6: XSS via postMessage
Module 7: Mutation XSS
Module 8: XSS in PDF generators
Module 9: Blind XSS
Module 10: XSS filter evasion
Module 11: Real-world XSS case studies
```

The skill writes a `research-plan.json` with one artifact spec per module. You review it, approve or tweak, and research starts. Foundations/techniques are NOT auto-added — your curriculum IS the decomposition. Use this to turn any paid-course outline into an open-source study pack.

### What happens after you hit enter

1. Claude writes `research-plan.json` — the list of documents it's going to produce. **Review it before letting it continue.** If a topic you care about isn't in there, say so now. Cheap to fix at planning, expensive later.
2. Claude starts on document 00 (foundations). This one is always the longest and most important. Watch this one.
3. After 00 passes, Claude starts 01 (techniques). Also watch.
4. After 01 passes, Claude may batch 02+ in parallel (3 at a time by default).
5. At the end, Claude runs a coherence check across all documents.
6. You get a folder of documents plus checklists and a concept map.

### When to interrupt

- If 00 comes out weird → stop, fix the plan, restart. 00 sets the voice for everything.
- If the judge keeps scoring the same criterion low → stop, the writer isn't learning. Check the brief.
- If you hit the 5-hour Opus limit → save state (the plan JSON + whatever documents passed), come back in a fresh session, tell Claude "resume from artifact N".

---

## 6. What you get out

```
{topic}-mastery/
├── research-plan.json         ← the plan and status
├── artifacts/
│   ├── 00-foundations.md      ← the longest, the "click" document
│   ├── 01-techniques.md
│   ├── 02-bypasses.md
│   ├── ...
│   ├── 07-case-study.md
│   └── briefs/
│       ├── 00-brief.md        ← scout's extraction (you rarely need these)
│       └── ...
├── judges/
│   ├── judge-00-loop1.md      ← score + critique per document per loop
│   └── ...
├── checklists/
│   ├── testing-checklist.md   ← things to try on a real target
│   └── spotting-guide.md      ← how to recognize the vuln in the wild
├── maps/
│   ├── concept-map.md         ← how all the pieces connect
│   └── concept-map.mermaid    ← diagram file
├── token-log.jsonl            ← how much each step cost
└── coherence-report.md        ← end-of-run cross-document check
```

**Read in this order:** 00 → checklists → 01 → pick the advanced docs that matter to you → 07 case study.

**`briefs/` and `judges/`** are work-product. You only read them if you're debugging why an artifact came out weird.

---

## 7. Token & context optimization — practical tips

These tips apply to v2 specifically, but most transfer to any long Claude session.

### 7.1 Context vs tokens — they're different

- **Tokens** = how much you pay. Every word in your conversation costs tokens. Claude has a daily token budget on the $20 plan.
- **Context window** = how much Claude can "see" at once. Once it fills up, Claude starts forgetting earlier parts or auto-compacts them. Bigger context = slower responses + higher cost per turn.

v2 attacks both. Scout offloads raw reading (saves tokens AND context). Brief-gate uses Haiku (saves tokens). Early-exit at score ≥85 (saves loop tokens). Paths-only-in-main (saves context).

### 7.2 Rules of thumb

1. **Don't paste long content into chat.** If you have a 40k-token document, save it to a file and tell Claude the path. Every time Claude reads the file it costs tokens, but the tokens don't stick in main context.
2. **Small prompt, big output > big prompt, small output.** Paying for input and output is symmetric, but huge inputs also eat context window.
3. **Prefer `Edit` over rewrite.** Claude rewriting a whole 3000-word artifact to fix 3 sentences costs 3000 output tokens. `Edit` costs ~200.
4. **Subagents are tokens-cheap if you brief them right.** A scout spawn that returns a 800-word brief saves you reading 30,000 words of RFC. Net win is massive. But a chatty scout that asks three clarifying questions costs more than it saves.
5. **Don't keep asking the same question in one session.** Claude re-reads the prompt each turn. If you've been in a session for 2 hours, the same question costs more than in a fresh session.
6. **Split sessions at natural boundaries.** After 00 and 01 are done, start a fresh session to do 02+. The new session reads 00/01 as exemplars and starts fresh.
7. **Monitor the token log.** `research/{topic}/token-log.jsonl` tells you where the tokens went. If scout is costing more than writer, something's wrong.
8. **Kill the loop early when the score is good.** v2 does this automatically at ≥85. If you see a 92 on loop 1, don't run loop 2 "just to be sure" — you're burning tokens on diminishing returns.

### 7.3 Specific v2 levers

| Lever | How | When |
|---|---|---|
| Fewer loops | `— 2 loops` | Lower stakes topic, or you're in a rush |
| Serial only | `— serial` | You've seen parallel runs drift in voice |
| Smaller batch | `— batch 2` | You're on a slow connection / hit rate limits |
| **Research depth** | `— depth light/standard/deep` | See §7.5 below |
| Haiku for brief-gate | default | Always leave it |
| Sonnet scout | default | Opus-scout wastes money |
| Opus writer | default | Sonnet-writer loses depth, don't change |
| Fresh session at doc 02 | manual | After doc 01 passes, `/clear` and resume with exemplars |

### 7.5 Research depth — the knob that matters most

Applies universally (any artifact, any topic, any mode). Controls how many sources scout reads per document.

| Depth | Searches | Fetches | Use for |
|---|---|---|---|
| `light` | 3 | 2 | Well-trodden topics (basic reflected XSS, basic CSRF). Saves ~40% of scout cost. |
| `standard` | 5–8 | 3–5 | Default. Covers 90% of what you need. |
| `deep` | 8–12 | 5–8 | Niche or esoteric (prototype pollution, cache deception, CSP bypass edge cases). |

**Strategy:** start the whole topic at `standard`. If the judge says "thin coverage" on loop 1 for a specific document, bump THAT document to `deep` and re-run only that one. Drop well-known documents to `light` so the budget you save goes to the hard ones.

**Per-artifact override:** in the plan JSON, any artifact can have its own `research_depth` field that wins over the topic-level default.

```json
{
  "topic": "XSS",
  "research_depth": "standard",
  "artifacts": [
    { "id": "01", "title": "Reflected XSS basics", "research_depth": "light" },
    { "id": "07", "title": "Mutation XSS", "research_depth": "deep" }
  ]
}
```

**Quality signal to watch:** after scout runs, the brief has `source_coverage: complete | partial`. If it's `partial` on an artifact that matters, re-spawn scout with `deep`. Don't let partial coverage ship.

### 7.4 What NOT to do

- Don't set `— 5 loops`. Past 3 loops, diminishing returns. If score isn't passing, the brief is bad, not the writing.
- Don't run `— batch 5`. Voice drift becomes visible. 3 is the sweet spot.
- Don't put raw RFCs in your prompt. Save them to `oauth-raw/*.md` files and point the scout at them.
- Don't have Claude "summarize" a document into context just to move on. Summaries are lossy AND still cost tokens. Trust the file on disk.
- Don't run v1 and v2 on the same topic simultaneously thinking you'll merge. Pick one.

---

## 8. The $20 plan survival guide

On the $20 plan you get a budget of Opus minutes every 5 hours. Running out mid-topic is painful.

### Survival rules

1. **Check context before starting a new document.** Use `/context` to see how full the window is. If it's over 60%, start a fresh session.
2. **Save state between sessions.** v2 writes `research-plan.json` + all artifacts to disk continuously. A new session resumes by reading the plan and checking which artifacts are marked complete.
3. **Don't idle in a big session.** Every turn in a full context costs more than the same turn in a fresh one. If you step away, close the session.
4. **Use v2, not v1, for new topics.** v1 is fine if you're committed to one topic and ready to spend. v2 fits the $20 plan much better.
5. **Use Haiku where you can.** v2 already does this for the two gates. If you add your own quick checks, use Haiku.
6. **Plan runs for dense sessions, not scattered ones.** A single 90-minute focused session does more than six 15-minute sessions because each fresh session pays a startup cost to read the plan + exemplars.
7. **If you hit the 5-hour limit mid-document:** note the document and loop number. Come back in a new session, tell Claude "resume artifact 03 at loop 2, judge report is at judges/judge-03-loop1.md". v2's file-on-disk discipline is designed for this.

### A realistic $20-plan workflow

- Session 1 (90 min): plan + documents 00, 01. These two set voice and foundation. Do them carefully, serial.
- Gap (wait for Opus budget refresh or step away).
- Session 2 (90 min): documents 02, 03, 04 in parallel batch. Coherence check at the end if budget allows.
- Session 3 (60 min): documents 05, 06, 07 + final coherence check + checklists.

Rough Opus budget per session: 00 is ~40% of a full budget, 01 is ~25%, each 02+ is ~15%. Parallel batches share the budget.

---

## 9. Common failure modes & how to spot them

| Symptom | Probable cause | Fix |
|---|---|---|
| Document 00 scores low on Foundation | Scout brief was paraphrased, not quoted | Re-spawn scout with "quote verbatim" emphasized |
| All documents sound the same topic-wise | Voice drift — writer lost the exemplar | Re-spawn writer with exemplar 00 in the prompt |
| Judge keeps asking for "more depth" on the same section | Must-include not covered in brief, writer padded | Re-run brief-gate, add the missing item to scout's re-spawn |
| Parallel documents contradict each other | Coherence judge didn't run OR human didn't act on report | Read `coherence-report.md`, apply fixes manually |
| Token log shows scout = 60% of run cost | Scout is reading too many sources | Tighten `sources_to_start` in the plan |
| Run blows the 5-hour limit on doc 03 | Too many loops or session too old | Fresh session, `— 2 loops`, resume from doc 03 |
| v2 output feels shallower than v1 | Brief truncation dropped nuance | Check for `scout_confidence: low` items, re-spawn scout |

---

## 10. Glossary

- **Artifact** — one study document. Foundations is artifact 00.
- **Brief** — scout's short extraction from the raw sources, consumed by the writer.
- **Appendix** — scout's overflow file for content that doesn't fit the brief's word cap.
- **Must-include** — the list of things every document must cover (from the plan).
- **Judge** — the rubric-based reviewer. Scores on 7 criteria, max 100.
- **Loop** — one judge + refine cycle.
- **Ralph loop** — the whole loop pattern (judge → refine → re-judge until PASS).
- **Early-exit** — v2's rule: score ≥85 and no MUSTs → stop, ship it.
- **Orchestrator** — the main Claude session.
- **Subagent** — a spawned Claude session with its own isolated memory.
- **Scout** — subagent that researches and writes briefs.
- **Writer** — subagent that writes the artifact from the brief.
- **Brief-gate / Pre-write gate** — Haiku checks that catch coverage problems early.
- **Coherence judge** — end-of-run pass that checks all documents together.
- **Exemplar** — a previously finalized artifact passed to later writers as a voice model.
- **7-criteria rubric** — Foundation /25, Signals /15, Req-Resp /20, Practitioner /15, Slop /10, Accuracy /10, Gaps /5. Total /100. PASS ≥75.

---

## Where to go next

- Read [SKILL.md](SKILL.md) for the orchestration spec.
- Read [AB-TEST.md](AB-TEST.md) if you want to compare v1 vs v2 side by side.
- Read [v2-proposal.md](../research%20skill/v2-proposal.md) for the design rationale and judge reports.
- Read [references/orchestrator.md](references/orchestrator.md) if you want to understand exactly what main does.

The skill is security-only by design. Both input modes (topic + curriculum) assume security material. If you ever need non-security research, that's a separate skill — don't bend this one.
