# V1 Foundation Pass: Access-Control Gadgets

Seed framing: Bugcrowd describes gadgets as small bugs or quirks that may be low severity alone, but become useful when saved and chained with another weakness. For access control, the gadget is rarely a magic payload. It is a boundary mistake: the app lets a caller influence an object, tenant, field, role, state, file, job, channel, or cache key that should have been bound to server-side authorization.

## Definition

An access-control gadget is a low-impact authorization primitive that becomes high impact when paired with an amplifier. The primitive might be "user A can see one canary record from user B", "a viewer can create a child comment on an object they cannot edit", or "a private file preview has a separate URL". The amplifier is what turns it into a reportable chain: export, webhook, async job, API token, role change, ownership transfer, GraphQL nested resolver, WebSocket subscription, cache/CDN storage, or legacy API route.

## WHY Model

Authorization is a tuple, not a yes/no check:

```text
allow(subject, action, object, field, tenant, state, channel, scope)
```

Most access-control bugs happen when the app checks only part of that tuple.

- BOLA/IDOR: the function is allowed, but the object is not.
- BOPLA: the object is allowed, but a field or writable property is not.
- BFLA: the object might be irrelevant because the function itself is not allowed.
- Tenant isolation: the object lookup is valid, but not scoped to the current tenant.
- Context-dependent access control: the action is valid in some state, but not now.
- Channel drift: REST is fixed, but export, WebSocket, GraphQL, mobile, cache, or async worker still uses older rules.

## Behavioral Signals Before Payloading

- Requests contain `user_id`, `account_id`, `org_id`, `tenant_id`, `workspace_id`, `project_id`, `file_id`, `job_id`, `report_id`, `invoice_id`, `role_id`, or `parent_id`.
- The UI loads list/search results first, then calls detail endpoints with IDs from the first response.
- A child object endpoint has both child and parent IDs, such as `/projects/{id}/comments/{comment_id}`.
- The app creates exports, reports, previews, thumbnails, transcodes, or async jobs and returns a second artifact ID.
- GraphQL exposes global IDs, `node`, `nodes`, nested objects, or mutation inputs with hidden fields.
- WebSocket messages contain `action`, `channel`, `room`, `project`, `tenant`, or object IDs.
- Responses include `X-Cache`, `Age`, `Cache-Control: public`, signed URLs, CDN hostnames, or static-looking derivative paths.
- Legacy routes, mobile APIs, old versions, admin helper routes, and undocumented endpoints use the same objects as modern routes.
- 403/404/count/timing differences reveal that an object exists even when content is blocked.

## Safe Testing Frame

Use two accounts you own, two roles, and ideally two tenants. Create canary objects with harmless marker text. Prove only the boundary break and the amplifier path. Do not enumerate. Do not touch real users. Revoke created tokens, webhooks, invites, shares, and test jobs.

## Sources

| URL | Used for | Tier |
|---|---|---|
| https://www.bugcrowd.com/blog/how-to-find-bugs-on-a-hardened-target-using-gadgets/ | Gadget mindset and chaining frame | web_fetch |
| https://owasp.org/API-Security/editions/2023/en/0x11-t10/ | API1/API3/API5/API6/API9 categories | web_fetch |
| https://cheatsheetseries.owasp.org/cheatsheets/Insecure_Direct_Object_Reference_Prevention_Cheat_Sheet.html | IDOR definition and object checks | web_fetch |
| https://portswigger.net/web-security/access-control | Horizontal, vertical, context-dependent access control | web_fetch |

