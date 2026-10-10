# Refine (v2)

Take the judge report. Fix every MUST. Address every SHOULD. Kill every slop. Fill gaps with NEW research — route research through Scout, not main context.

---

## v2 routing rule (new)

| Situation | Path | Who does it |
|---|---|---|
| ≤3 MUSTs, all additive (add a sentence, add a blocked case, rewrite a slop line) | Targeted `Edit` calls | Main orchestrator |
| ≥4 MUSTs, OR structural change, OR section reorder | Spawn writer in refine mode | Writer subagent (Opus) |
| MUST needs new research (e.g., "cite the real Storm-XXXX campaign") | Spawn scout in gap-only mode first, then writer | Scout (Sonnet) → Writer (Opus) |

**Never read the full artifact into main for refine.** Either `Edit` specific ranges, or hand the whole job to a writer subagent. Main sees judge report + file paths, not artifact body.

---

## Process (per v1, applies inside the writer subagent in refine mode)

1. **Read judge report.** Categorize: MUST / SHOULD / slop / missing.

2. **Research gaps (token-efficient):**
   - Tier 1 first: targeted search for the specific gap ("DNS rebinding SSRF mechanism")
   - Tier 2 only if blocked. Tier 3 almost never during refinement.
   - Budget: 2-3 searches, 1-2 fetches per refine pass.
   - Add new URLs to Sources table with "Added Loop N"

3. **Rewrite:**
   - Preserve structure unless judge said otherwise
   - Preserve what judge marked as good — don't paraphrase into worse
   - Fix in place — don't shuffle content
   - Add missing content where it logically fits, not a "bonus" section at the end
   - Every slop line gets REWRITTEN, not deleted

4. **Clarity techniques:**

   **Analogy bridge:** "DNS rebinding is like checking ID at the door, but by the time they sit down they've swapped faces. Filter checks IP at resolution (door), attacker's DNS returns DIFFERENT IP when request is made (face swap)."

   **"Why not" questions:** "Why `{{7*7}}` not `{{7+7}}`? Both work. But `49` in a response is unmistakable — no innocent reason for it. `14` could already be on the page. Multiplication is a better canary."

   **Config alongside payload:** Show vulnerable code THEN the attack. Cause and effect together.

   **Error messages are gold:** "`URL scheme must be http or https` → filter validates scheme, SSRF works, stay within http. `Connection refused` → SSRF works, wrong port, try 80/443/8080/6379/27017."

5. **Self-check:**
   - [ ] Every MUST fixed
   - [ ] Every slop rewritten
   - [ ] Gaps filled with new research (or flagged unverifiable)
   - [ ] No NEW slop introduced
   - [ ] Reads as a whole, not patchwork
   - [ ] All techniques have request + success + failure

Overwrite previous version. Update research-plan.json.
