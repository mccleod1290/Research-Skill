# Pre-write outline judge (Haiku)

> One question. One deterministic output. No rubric overlap with the main judge.

Runs BEFORE the writer spawns. Catches missing sections before 2000 words of writing happen.

---

## Input

- `outline`: 10-line structural sketch main produced from the brief. Each line is one section heading + one-phrase purpose.
- `must_include`: array from `research-plan.json` for this artifact.

---

## The only question

> For each `must_include[k]`, does at least one outline line clearly cover it?

Do NOT score depth, quality, slop, or accuracy. Those are the main judge's job, on the finished artifact, against the 7-criteria rubric in `judge.md`.

---

## Output format (strict)

Either:

```
PASS
```

Or:

```
FAIL
must_include[1] "PKCE verifier mismatch blocked response" → add section covering this, suggested placement: after §4 PKCE.
must_include[4] "client_secret leak checklist" → no outline line covers this.
```

That's it. No commentary, no summary. Main consumes the output programmatically.

---

## Why Haiku

- Single-step mapping: array → array. No chain of reasoning.
- Deterministic output schema: PASS or FAIL+list.
- Cost: ~300 tokens in, ~50 out. Running on Sonnet or Opus would be waste.

---

## What this does NOT do

- Score the outline.
- Suggest content for sections.
- Critique ordering, voice, or style.
- Overlap with any of: Foundation / Signals / Req-Resp / Practitioner / Slop / Accuracy / Gaps.

If main wants substantive outline feedback, that's a separate call. This gate only checks coverage.
