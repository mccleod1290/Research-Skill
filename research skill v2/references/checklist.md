# Checklist Builder

Read ALL finalized artifacts. Produce two files.

---

## 1. Testing Checklist → `checklists/testing-checklist.md`

```markdown
# [Topic] Testing Checklist

## Phase 1 — Recon
- [ ] **[Signal]** — [where to look, what it means]

## Phase 2 — Detection
- [ ] **[Test]** — `[payload]` → expect: [what success looks like]

## Phase 3 — Exploitation
- [ ] **[Step]** — `[payload]` → if blocked: `[bypass]`

## Phase 4 — Escalation
- [ ] **[Chain]** — from [this vuln], try [next]

## Bypass Quick Ref
| Blocked by | Try |
|---|---|
| [defense] | `[bypass]` |
```

Rules: checkbox on every item, payload in backticks, success described, ordered least→most invasive, bypasses inline.

---

## 2. Spotting Guide → `checklists/spotting-guide.md`

```markdown
# [Topic] Spotting Guide

## App Behaviors
- [ ] **[Behavior]** — [why it suggests the vuln]

## Suspicious Parameters
- [ ] `?url=` / `?fetch=` — [what each suggests]

## Error Messages
| You see | Means | Try |
|---|---|---|
| `[error]` | [interpretation] | `[next payload]` |

## Vulnerable Code Patterns
```[lang]
// [code with comments on which line is the problem]
```
```

Quality check: every payload from every artifact appears in a checklist.
