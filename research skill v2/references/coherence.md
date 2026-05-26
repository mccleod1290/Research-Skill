# Cross-artifact coherence judge (Sonnet, one pass, end of run)

> Read shallow. Flag inconsistencies. Do NOT re-score. Do NOT rewrite.

Runs after ALL artifacts are PASS. One Sonnet subagent call. Reads:
- Section headers from every artifact
- "Keep" sections from every artifact's final judge report
- `must_include` arrays from `research-plan.json`

Does NOT read the full artifact bodies. That's the point — it's a coherence scan, not a re-judgment.

---

## What to flag

### 1. Contradictions
Artifact A says X is deprecated. Artifact B recommends X without caveat. Flag both locations.

### 2. Unclaimed must_includes
A topic was promised in `research-plan.json` for artifact N but no section heading or judge "Keep" note mentions it across the whole set.

### 3. Heavy duplication
Two artifacts both have a 500+ word section on the same concept with overlapping examples. Flag which to prune or cross-reference.

### 4. Voice drift
One artifact uses a noticeably different tone (academic, marketing-flavored, hedge-y) from the others. Flag the artifact ID + one example phrase.

### 5. Broken cross-references
Artifact 03 says "see artifact 05 for PKCE downgrade" but artifact 05's headers don't contain that section.

---

## Output

Write `research/{topic}/coherence-report.md`:

```markdown
# Coherence report — {topic}

## Contradictions
- [A says X, B says Y, where] → [suggested resolution]

## Unclaimed must_includes
- `must_include[k]` on artifact N, not covered anywhere

## Duplications
- Concept X — artifact 03 §2 and artifact 05 §4, ~400 words each → suggest prune in 05, link to 03

## Voice drift
- Artifact 04 — reads more formal than 00–03. Example: "It should be noted that..."

## Broken refs
- Artifact 03 → Artifact 05 §PKCE downgrade: target section does not exist

## No issues found
- [List sections that are clean]
```

Return path to main. Done.

---

## What this does NOT do

- Run the 7-criteria rubric. That already happened per-artifact.
- Rewrite artifacts. Human decides fixes.
- Score a total. No score.
- Read full artifact bodies — too expensive, not needed for coherence.

---

## When main acts on the report

Human reads `coherence-report.md`. Chooses which fixes to apply. For each chosen fix:
- Small edit → main does `Edit` call directly.
- Structural change → re-spawn writer for that artifact in refine mode.
- Re-judge the changed artifact if the edit touched scored content.

Coherence pass itself never loops. Runs once.
