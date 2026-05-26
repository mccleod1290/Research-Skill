---
name: research-mastery
description: >
  Deep research skill for mastery-level study material on ANY security topic (SSRF, SSTI, XXE, IDOR, OAuth, etc). Trigger: "research this", "build mastery on", "deep dive", "ralph loop on", "research stack on". Produces artifacts with WHY-level depth, testing checklists, concept maps. Uses 3-tier web research (fetch → curl → browser) and Ralph loops (judge → refine → repeat).
---

# Research Mastery

> Topic in. Practitioner-grade study material out.

```
"Use research-mastery to build mastery on SSRF with 3 Ralph loops"
"Research SSTI deeply — 2 loops"
"Deep dive into XXE"  (defaults to 3 loops)
```

---

## Step 1 — Plan the Artifacts

Decompose the topic by asking 5 questions:

1. **Distinct bypass categories?** → bypass compendium artifact
2. **Protocol/chain variants?** (SSRF→Gopher→RCE) → chain artifact
3. **Platform-specific variations?** (AWS/GCP/Azure, Jinja2/Twig/Freemarker) → platform artifact(s)
4. **Blind/out-of-band variant?** → blind detection artifact
5. **Famous real-world case study?** → case study artifact

Every topic gets **00-foundations** and **01-techniques** automatically. Questions 1-5 add 02+. Minimum 5 artifacts, maximum 10.

**Artifact 00 is always the longest.** It's the one that makes everything else click.

Output `research-plan.json`:
```json
{
  "topic": "SSRF",
  "loops": 3,
  "artifacts": [
    {
      "id": "00",
      "title": "Why Servers Fetch What You Tell Them",
      "filename": "00-foundations.md",
      "focus": "Root cause, vulnerable code patterns, behavioral signals, server configs, trust boundary",
      "sources_to_start": ["PortSwigger SSRF", "OWASP SSRF", "HackTricks SSRF"],
      "must_include": ["vulnerable code pattern", "server config", "behavioral signals list", "trust boundary explanation"],
      "min_words": 1500,
      "status": "pending",
      "loop": 0,
      "score": null,
      "judge_notes": []
    }
  ]
}
```

### Example: SSRF decomposition

```
00 - Foundations: root cause, code patterns, configs, behavioral signals
01 - Core techniques: basic SSRF, IP formats, response interpretation
02 - Blind SSRF: no response, DNS/HTTP callbacks, timing
03 - Cloud metadata: AWS/GCP/Azure endpoints, IMDSv1 vs v2, token theft
04 - Filter bypass compendium: IP encoding, DNS rebinding, URL parser tricks, redirects
05 - Gopher protocol chains: Gopher→Redis, Gopher→SMTP, protocol chains to RCE
06 - URL parser differentials: Orange Tsai, when validator and fetcher disagree
07 - Case study: a real SSRF bounty report dissected
```

### Example: SSTI decomposition

```
00 - Foundations: why template engines execute input, how rendering works,
     why {{7*7}}=49, why multiplication > addition as canary,
     what the parser DOES with expressions vs static text
01 - Detection + fingerprinting: polyglots, engine ID decision tree,
     why ${} vs {{}} — different delimiters, different expression languages
02 - Jinja2/Python chains: MRO traversal, __subclasses__, sandbox escapes
03 - Twig/PHP chains
04 - Freemarker/Java chains
05 - Filter bypass + WAF evasion
06 - Blind SSTI detection
07 - Case study
```

---

## Step 2 — Research + Write (per artifact)

Read `references/writer.md`. Covers 3-tier research escalation and all writing rules.

### The 3 Research Tiers

**Tier 1 — web_search + web_fetch (START HERE)**
Search first. If snippet answers the question, don't fetch the page. Budget: 5-8 searches, 3-5 fetches per artifact.

**Tier 2 — curl with browser headers (when Tier 1 gets blocked)**
```bash
curl -s -L \
  -H "User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/125.0.0.0 Safari/537.36" \
  -H "Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8" \
  -H "Accept-Language: en-US,en;q=0.5" \
  -H "Connection: keep-alive" \
  "URL" 2>/dev/null | head -500
```
Budget: 2-3 per artifact.

**Tier 3 — Playwright / browser automation (LAST RESORT)**
Only for JS-rendered content. Budget: 0-1 per artifact.

**Stop rule:** If Tier 1 gives enough for a solid draft, STOP. The judge finds gaps — research them in the refine loop.

---

## Step 3 — Judge (per artifact)

Read `references/judge.md`. Scores on 7 criteria (/100).

≥75 = PASS → next artifact. 50-74 = REVISE → refine loop. <50 = REWRITE.

---

## Step 4 — Refine (per artifact that didn't pass)

Read `references/refine.md`. Takes judge critique, does TARGETED new research (Tier 1 first), rewrites.

---

## Step 5 — Ralph Loop

Repeat Steps 3-4 for specified number of loops. Process artifacts ONE AT A TIME — finish 00 through all loops before starting 01.

```
Loop 1: Judge scores 55 → Refine (adds DNS rebinding explanation, 3 new payloads)
Loop 2: Judge scores 72 → Refine (adds error message section, WAF bypass)
Loop 3: Judge scores 81 → PASS
```

---

## Step 6 — Final Outputs

Once ALL artifacts pass:

1. Read `references/checklist.md` → testing checklist + spotting guide
2. Read `references/concept-map.md` → concept map + Mermaid diagram + learning path

No web research needed. Reads finalized artifacts only.

---

## Output Structure

```
{topic}-mastery/
├── research-plan.json
├── artifacts/
│   ├── 00-foundations.md
│   ├── 01-techniques.md
│   └── ...
├── checklists/
│   ├── testing-checklist.md
│   └── spotting-guide.md
└── maps/
    ├── concept-map.md
    └── concept-map.mermaid
```

---

## Rules (apply to everything)

**WHY before WHAT.** Every technique explains why it works at the parser/runtime level. `{{7*7}}` returns 49 because Jinja2's parser treats `{{ }}` as expression delimiters and passes content to Python's `eval()`. THAT level.

**Configs that create the vuln.** The Flask route with `requests.get(user_input)`. Not "SSRF is possible."

**Behavioral signals.** What app behavior suggests this BEFORE payloads? URL inputs, PDF generators, webhooks, error messages.

**Real requests, real responses.** Full HTTP. Success. Failure. What to try next.

**Source tracking.** Every artifact ends with a table: URL, what was used, which tier.

**No slop.** Banned: comprehensive, robust, utilize, leverage, cutting-edge, "it's important to note." If removing a paragraph loses nothing specific, cut it.

**Explain at the bar.** Senior hacker to smart junior. Not a textbook.

---

## Reference Files

| File | Read When |
|---|---|
| `references/writer.md` | Step 2 — researching and writing |
| `references/judge.md` | Step 3 — critiquing |
| `references/refine.md` | Step 4 — improving |
| `references/checklist.md` | Step 6 — testing checklists |
| `references/concept-map.md` | Step 6 — concept map |
