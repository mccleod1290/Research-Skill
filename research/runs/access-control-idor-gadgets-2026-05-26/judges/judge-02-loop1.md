# Judge - Access-Control and IDOR Gadget Artifact - Loop 1

**Score:** 91/100 -> PASS

| Criterion | Score | Issue |
|---|---:|---|
| Foundation | 23/25 | Strong tuple model. Could add more case studies in a longer run. |
| Signals | 15/15 | Signals are practical and cover the requested surfaces. |
| Req/Resp | 17/20 | Safe generic HTTP examples included; target-specific examples intentionally omitted. |
| Practitioner | 15/15 | Gadget table and chain formulas are usable during authorized testing. |
| Slop | 10/10 | Banned wording avoided. |
| Accuracy | 10/10 | Grounded in OWASP, PortSwigger, and Bugcrowd sources. |
| Gaps | 1/5 | Could later add public bounty case studies for each category. |

## MUST Fix

None.

## SHOULD Fix

1. Add a case-study appendix if the user wants a longer v3 hunt.

## Keep

- Authorization tuple framing.
- Canary-only safety model.
- Channel drift as a unifying source of gadgets.
- Report wording section.

