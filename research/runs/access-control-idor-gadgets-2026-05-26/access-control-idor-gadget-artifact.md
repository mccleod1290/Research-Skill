# Access-Control and IDOR Gadget Artifact

Audience: authorized bug bounty testing with accounts, tenants, and canary data you control.

This artifact uses Bugcrowd's gadget framing: a gadget is a small bug, misconfiguration, or strange behavior that may be low severity alone, but becomes high impact when chained with another weakness. For access control, the gadget is usually a broken boundary that can feed an amplifier such as export, async job, webhook, token creation, role change, file derivative, GraphQL resolver, WebSocket channel, cache/CDN path, or legacy API route.

## 1. Definition: Access-Control Gadget

An access-control gadget is a low-impact authorization primitive that exposes or influences one part of an authorization decision:

```text
allow(subject, action, object, field, tenant, state, channel, scope)
```

The primitive alone might only prove "this object exists", "this child record accepts input", "this preview URL is separate", or "this search result leaks one canary title". The chain becomes serious when that primitive feeds a gadget with blast radius, persistence, privilege, or sensitive side effects.

Think in boundaries:

- Object boundary: can this subject act on this exact object?
- Tenant boundary: does the object belong to the active tenant or org?
- Parent/child boundary: does access to a child require access to the parent?
- Property boundary: can this role read or write this specific field?
- Function boundary: can this role call this endpoint or action at all?
- State boundary: is this action allowed in the object's current workflow state?
- Channel boundary: does the same policy hold in REST, GraphQL, WebSocket, export, async worker, file service, cache, and mobile API?
- Scope boundary: does the token, invite, share, webhook, OAuth grant, or integration have the right audience and limits?

The best gadgets do one of four things:

- Add blast radius: one-object leak -> bulk export, report, search index, webhook stream.
- Add privilege: low role -> role mutation, invite, admin helper route, function-level bypass.
- Add persistence: temporary access -> API token, OAuth app, share link, ownership transfer.
- Add side effect: read-only flaw -> workflow transition, file operation, notification, billing change.

## 2. Behavioral Signals Before Payloading

Before sending test payloads, collect behaviors. You are looking for places where the app tells you what boundary it trusts.

### Request and URL Signals

- IDs in paths: `/users/{id}`, `/orgs/{org_id}/projects/{project_id}`, `/files/{file_id}`.
- IDs in bodies: `user_id`, `owner_id`, `tenant_id`, `account_id`, `workspace_id`, `parent_id`, `role_id`, `invoice_id`, `job_id`.
- IDs in headers: `X-Org-ID`, `X-Tenant-ID`, `X-Workspace`, `X-Account`, custom mobile headers.
- Mixed parent and child IDs in one request.
- Body contains fields the UI did not expose.
- Same operation supports several methods, such as `GET`, `POST`, `PATCH`, `DELETE`, or `_method`.
- Error differences: `403` vs `404`, count changes, "not in workspace", "invalid state", "already exists", or faster/slower response.

### UI and Workflow Signals

- UI hides a button, but the network request is predictable.
- A role can view an object but not edit it; the API still loads edit metadata.
- A multi-step flow has `prepare`, `preview`, `confirm`, `approve`, `finalize`, or `complete` endpoints.
- Review, approval, payout, publish, KYC, email change, MFA reset, or invite flows rely on client-side steps.
- Audit logs or notifications reveal object IDs, actor IDs, tenant names, or workflow states.

### Secondary Channel Signals

- Export, PDF, CSV, report, print, dashboard, or saved-search endpoints reuse list filters.
- Async jobs return `job_id`, `artifact_id`, `download_url`, `status_url`, or `result_url`.
- File services create thumbnails, previews, OCR text, transcodes, watermarked copies, or signed URLs.
- Search and autocomplete return snippets, counts, facets, owner names, emails, or IDs.
- GraphQL exposes `node`, `nodes`, global IDs, nested objects, or mutations with wide input types.
- WebSocket messages contain `action`, `room`, `channel`, `project`, `tenant`, or object IDs.
- CDN/cache responses include `X-Cache`, `Age`, `Via`, `CF-Cache-Status`, `Cache-Control: public`, or static-looking paths.
- Mobile, legacy, beta, internal, admin helper, or versioned APIs duplicate modern functionality.

## 3. Categorized Gadget List

Use this as a hunting map. Each category lists the boundary, why it amplifies impact, and a safe canary proof.

| Gadget category | Boundary to test | Why it amplifies severity | Safe proof pattern |
|---|---|---|---|
| Object detail read | Object | Base IDOR/BOLA primitive; proves direct object authorization is missing | Two owned accounts; account B creates canary object; account A requests only that known ID |
| Object update/delete | Object + action | Turns read issue into integrity or availability impact | Use a disposable canary object; show unauthorized edit is accepted or blocked inconsistently |
| Tenant context switching | Tenant | Cross-tenant access is usually higher impact than same-tenant horizontal access | Two owned tenants; swap only tenant or workspace context; verify canary isolation |
| Parent/child object read | Parent + child | Child records often skip parent authorization | Try comments, notes, tasks, attachments, approvals, messages, labels on canary parent objects |
| Child object creation | Parent + action | Low write can influence parent workflow, audit trail, notifications, or approvals | Add harmless note/tag/comment to a foreign canary parent and check visibility or side effects |
| Search/list/filter | Object + tenant | Reveals IDs and metadata that feed stronger gadgets | Search for unique canary marker from another account; prove count/title/snippet only |
| Autocomplete/lookup | Object existence | Leaks users, emails, orgs, projects, billing entities, or invite targets | Query exact canary name/email; avoid broad enumeration |
| Saved report/dashboard | Object + field + tenant | Saved filters may execute with owner or system context | Share or execute a report containing only canary data from the wrong role |
| Export/PDF/CSV | Object + field + job | Converts one-object weakness into bulk disclosure | Export one canary row; check job creation and download authorization separately |
| Async job status/result | Job owner + target object | Workers often run with stale or elevated context | Queue job with owned canary IDs; verify status/result cannot cross accounts or tenants |
| Bulk action/import | Per-item object | Batch endpoints may check only first item or job owner | Submit one valid canary and one unauthorized canary; expect explicit per-item rejection |
| File download | Parent object + file | Storage keys/CDN URLs can outlive parent permission | Upload benign marker file; test whether another controlled account can fetch by file ID |
| File derivative | Parent object + derivative | Preview/thumbnail/OCR/transcode paths often have separate ACLs | Use harmless image/PDF marker; test derivative URL, not real private files |
| Signed URL reveal | Scope + expiry | Signed URLs can turn a minor metadata leak into direct file access | Verify URL audience, expiry, and revocation with test files only |
| Cache/CDN private response | Cache key + auth context | Private data may be served without the session that generated it | Use canary account page and cache headers; never trick real users into priming cache |
| GraphQL node/global ID | Resolver object | `node`/`nodes` can bypass edge-scoped checks | Use canary global ID; compare access through relation path vs direct node path |
| GraphQL nested field | Resolver field | Parent resolver checks pass, nested resolver leaks fields | Request only harmless canary fields first; test field-level denial consistency |
| GraphQL mutation input | Function + property | Wide input types can accept hidden fields | Use disposable canary object; add protected field with harmless value and verify rejection |
| WebSocket subscription | Channel + tenant | Live channels leak events outside REST controls | Subscribe to a canary room/project; trigger synthetic event from owned account |
| WebSocket action message | Function + object + state | Connection auth may replace per-message auth | Send benign action against canary object; expect message-level denial |
| Notification settings | Recipient + object | Out-of-band leak through email, Slack, SMS, push, inbox | Route canary notification to controlled destination only |
| Webhook configuration | Event source + destination | Turns read bug into continuous data stream | Configure webhook for owned endpoint; subscribe only to canary object events |
| Audit/activity log | Tenant + role + immutability | Logs leak object IDs, users, workflow state, tokens, or evidence | Read only canary events; never alter real audit records |
| Invite creation | Function + tenant + role | Persistent org access and vertical escalation | Use owned tenant; verify low role cannot invite or choose elevated role |
| Invite redemption | Token + recipient + tenant | Weak binding lets invite cross accounts or roles | Use two controlled emails; verify token is bound to intended recipient and tenant |
| Role/group assignment | Function + property | Direct privilege escalation | Try harmless role field on disposable member; stop at proof of acceptance/rejection |
| Ownership transfer | Object + durable control | Converts temporary access into long-term control | Transfer only disposable objects between owned accounts |
| Share/public link | Object + audience + expiry | Expands audience and may bypass normal ACLs | Create test share; verify viewer/editor scope, expiry, and revocation |
| API token creation | Function + scope | Persistence after the original weak request | Create/reveal/revoke only test tokens; verify scope cannot exceed caller |
| OAuth app install/consent | Delegated scope + tenant | Long-lived delegated access and org-wide install risk | Use a test app and dummy scope; confirm low roles cannot grant org scopes |
| Integration mapping | Tenant + downstream identity | Connector may run as stronger service account | Map only test workspace; verify connector cannot read/write foreign canary object |
| Billing/invoices | Account + role + field | PII and financial operations raise impact | Use sandbox billing or non-charging canary invoice metadata |
| Plan/quota/feature flag | Entitlement | Client-selected entitlements can unlock admin/paid flows | Change only harmless test flags; verify server derives entitlement |
| Account recovery/email/MFA reset | Identity + state + token | Partial access can become account takeover | Use owned accounts; verify token purpose, recipient, expiry, and step-up |
| Support impersonation | Function + target + audit | High impact if callable by normal users | Never target real users; prove route rejects normal role or accepts only dummy target |
| Admin/helper/internal route | Function | Hidden UI is not authorization | Access only benign test endpoints; show missing role gate without destructive action |
| Legacy/mobile API | Inventory + channel | Old versions often skip newer checks | Compare modern and legacy route using same canary object |
| Method override | Function + method | `GET`/`POST`/`DELETE` drift can bypass UI assumptions | Try method changes on disposable canary object only |
| Workflow transition | State + role | Skip submit/review/approve/pay/publish sequence | Use draft canary object; direct-call later step and expect invalid-state denial |
| Race/replay/idempotency | State + atomicity | Single-use invites, approvals, coupons, and grants may double-spend | Repeat only harmless canary action; show duplicate state or duplicate event |
| Response overexposure | Field | API returns hidden properties the UI masks | Compare roles; document only canary secret field, not real secrets |
| Mass assignment | Property write | Hidden fields like `owner`, `role`, `status`, `verified`, `price` change authority | Submit protected field on canary object; stop after observing accepted change |

## 4. Safe Chain Formulas

These are chain shapes, not target instructions. Replace every object with canary data you own.

### Formula A: Metadata -> Object -> Export

```text
search/list gadget -> one canary object ID -> object read -> export/report artifact
```

Why it works: list/search endpoints often prove object existence or leak IDs. Export endpoints then multiply the impact if they run the same query without the same authorization filter.

Safe evidence:

```http
GET /api/search?q=CANARY-TENANT-B-MARKER HTTP/1.1
Host: authorized-target.example
Cookie: account_A_session
```

Expected secure result:

```http
HTTP/1.1 200 OK
{"results":[],"total":0}
```

Report impact phrasing: "The search endpoint exposes cross-tenant canary metadata and the export job preserves that unauthorized scope, raising impact from single-record discovery to bulk data retrieval."

### Formula B: Child Write -> Parent Workflow

```text
child create gadget -> parent object side effect -> notification/audit/workflow influence
```

Why it works: developers often authorize the child route but forget to re-check the parent object. A comment, approval note, attachment, tag, or status reason can affect parent state.

Safe evidence:

```http
POST /api/projects/{tenant_B_project}/comments HTTP/1.1
Host: authorized-target.example
Cookie: account_A_session
Content-Type: application/json

{"body":"CANARY unauthorized child write"}
```

Expected secure result:

```http
HTTP/1.1 403 Forbidden
{"error":"Access denied"}
```

Report impact phrasing: "Authorization is enforced on comment creation as a standalone function, but not on the parent project relationship."

### Formula C: File Metadata -> Derivative URL

```text
attachment metadata leak -> preview/thumbnail/OCR URL -> signed/CDN artifact
```

Why it works: file systems often split metadata, original file, preview, thumbnail, OCR text, and CDN signing into different services. One service may inherit less context than the parent object.

Safe evidence:

```http
GET /api/files/{tenant_B_canary_file}/preview HTTP/1.1
Host: authorized-target.example
Cookie: account_A_session
```

Expected secure result:

```http
HTTP/1.1 403 Forbidden
Cache-Control: no-store
```

Report impact phrasing: "The derivative file endpoint does not inherit the parent object's ACL, so a private attachment can be accessed through its preview path."

### Formula D: Tenant Confusion -> Async Job

```text
client-controlled tenant/workspace ID -> job enqueue -> worker reads target -> result URL
```

Why it works: job creation may check the caller, while the worker later trusts queued IDs. Status and result endpoints add more places for owner checks to drift.

Safe evidence:

```http
POST /api/reports/jobs HTTP/1.1
Host: authorized-target.example
Cookie: account_A_session
Content-Type: application/json

{"tenant_id":"tenant_B","report_id":"CANARY_REPORT_B"}
```

Expected secure result:

```http
HTTP/1.1 403 Forbidden
{"error":"Tenant scope mismatch"}
```

Report impact phrasing: "The job queue accepts a cross-tenant report reference and the worker processes it outside the request-time tenant context."

### Formula E: GraphQL Relation -> Direct Node

```text
authorized relation path denied -> direct node/global ID path allowed -> nested field leak
```

Why it works: GraphQL often has separate resolvers for edges, nodes, and nested fields. Authorization on the relationship does not prove authorization on the node resolver.

Safe evidence:

```http
POST /graphql HTTP/1.1
Host: authorized-target.example
Cookie: account_A_session
Content-Type: application/json

{"query":"query($id: ID!){ node(id:$id){ id ... on Project { name } } }","variables":{"id":"CANARY_GLOBAL_ID_B"}}
```

Expected secure result:

```http
HTTP/1.1 200 OK
{"data":{"node":null},"errors":[{"message":"Access denied"}]}
```

Report impact phrasing: "The edge resolver correctly hides the foreign project, but the global node resolver returns the same project when addressed directly."

### Formula F: WebSocket Subscribe -> Synthetic Event Leak

```text
channel join gadget -> controlled event trigger -> cross-tenant live update
```

Why it works: the handshake authenticates the user, but each message still needs authorization for action, object, tenant, and channel.

Safe evidence:

```json
{"action":"subscribe","tenant_id":"tenant_B","channel":"project:CANARY_PROJECT_B"}
```

Expected secure result:

```json
{"type":"error","code":"access_denied"}
```

Report impact phrasing: "The WebSocket server authorizes the connection but not the per-message subscription target."

### Formula G: Cache Key Drift -> Private Canary Response

```text
private dynamic response -> static-looking path/cache rule -> cache hit without auth context
```

Why it works: a CDN or reverse proxy may cache based on path, extension, or normalized URL while the origin treats the request as a dynamic resource.

Safe evidence:

```http
GET /account/canary-profile/test.css?cachebust=controlled1 HTTP/1.1
Host: authorized-target.example
Cookie: account_A_session
```

Expected secure result:

```http
HTTP/1.1 200 OK
Cache-Control: private, no-store
X-Cache: dynamic
```

Report impact phrasing: "The cache key omits authorization context and stores a private dynamic response under a static cache rule."

### Formula H: Legacy API -> Modern Object

```text
modern route denies -> old/mobile route accepts same canary object ID -> stronger action
```

Why it works: API inventory drift leaves old routes using pre-migration authorization logic.

Safe evidence:

```http
PATCH /api/v1/mobile/projects/{tenant_B_project} HTTP/1.1
Host: authorized-target.example
Cookie: account_A_session
Content-Type: application/json

{"name":"CANARY harmless rename attempt"}
```

Expected secure result:

```http
HTTP/1.1 403 Forbidden
```

Report impact phrasing: "The current web API denies cross-tenant access, but the legacy mobile endpoint performs the same action without the tenant-scoped policy."

## 5. Report Wording Hints

### Titles

- "Broken object authorization on report export exposes cross-tenant canary records"
- "Preview endpoint bypasses parent file ACL for private attachments"
- "GraphQL global node resolver bypasses project membership checks"
- "WebSocket subscription lacks per-message tenant authorization"
- "Legacy mobile endpoint allows unauthorized status transition on project objects"
- "Role assignment accepts protected `role_id` from non-admin member update"

### Impact Phrasing

Use chain language that shows why the gadget matters:

```text
The base issue is not identifier predictability. The server accepts a valid object reference without proving that the current subject has a relationship to that object.
```

```text
The export gadget raises impact because the same authorization miss applies to a bulk artifact, not just a single detail view.
```

```text
The preview service appears to authorize the derivative file separately from the parent document, so parent ACL changes do not protect the generated artifact.
```

```text
The WebSocket channel trusts the authenticated connection but does not re-check the tenant and object for each subscribe/action message.
```

```text
The legacy endpoint demonstrates policy drift: the modern route rejects the canary object, while the older route accepts the same action.
```

### Repro Structure

Keep triage friendly and safe:

1. State authorization scope: "All testing used two researcher-owned accounts and two researcher-owned tenants."
2. Name the canary: "Tenant B object contains marker `CANARY-AC-GADGET-001`."
3. Show the expected secure behavior from the UI or modern route.
4. Show the gadget request with only the canary ID.
5. Show the amplifier: export, job result, derivative URL, webhook, GraphQL resolver, WebSocket event, cache hit, or legacy route.
6. Explain cleanup: token revoked, webhook removed, share deleted, job canceled, canary object deleted.

### Evidence Matrix

| Evidence item | What it proves |
|---|---|
| Account/tenant diagram | The subjects and objects are owned test data |
| Baseline allowed request | Account B can access its own canary |
| Baseline denied request | Account A should not access tenant B |
| Gadget request | The specific boundary that fails |
| Amplifier request | Why severity increases |
| Server response | Actual behavior with status code and body |
| Cleanup proof | No persistent test access remains |

### Phrases to Avoid

- Avoid "I enumerated users" if you used a single known canary ID.
- Avoid "UUIDs are guessable" unless you actually proved discovery. The bug is missing authorization, not the ID format.
- Avoid claiming account takeover unless the chain reaches account control in owned accounts.
- Avoid reporting a gadget alone as high severity without the amplifier evidence.
- Avoid destructive proof. Use denied/accepted state, harmless canary field, or reversible test object.

## Minimal Testing Matrix

Run this with accounts and tenants you control.

| Axis | Account A low role | Account B same role | Account C admin | Tenant X | Tenant Y |
|---|---|---|---|---|---|
| Read detail | own canary | other user canary | admin canary | same tenant | cross tenant |
| List/search | own marker | other marker | admin marker | same tenant | cross tenant |
| Child create | own parent | other parent | admin parent | same tenant | cross tenant |
| Export/job | own data | other data | admin data | same tenant | cross tenant |
| File derivative | own file | other file | admin file | same tenant | cross tenant |
| GraphQL node | own global ID | other global ID | admin global ID | same tenant | cross tenant |
| WebSocket channel | own channel | other channel | admin channel | same tenant | cross tenant |
| Legacy/mobile | own object | other object | admin object | same tenant | cross tenant |
| Role/invite/share | own allowed action | disallowed action | admin action | same tenant | cross tenant |

## Source Table

| # | URL | Used for | Tier |
|---|---|---|---|
| 1 | https://www.bugcrowd.com/blog/how-to-find-bugs-on-a-hardened-target-using-gadgets/ | Gadget definition and chaining mindset | web_fetch |
| 2 | https://owasp.org/API-Security/editions/2023/en/0x11-t10/ | API access-control taxonomy | web_fetch |
| 3 | https://owasp.org/API-Security/editions/2023/en/0xa1-broken-object-level-authorization/ | Object-level authorization model | web_fetch |
| 4 | https://owasp.org/API-Security/editions/2023/en/0xa3-broken-object-property-level-authorization/ | Property-level reads/writes and mass assignment | web_fetch |
| 5 | https://owasp.org/API-Security/editions/2023/en/0xa5-broken-function-level-authorization/ | Function-level authorization and admin route drift | web_fetch |
| 6 | https://cheatsheetseries.owasp.org/cheatsheets/Insecure_Direct_Object_Reference_Prevention_Cheat_Sheet.html | IDOR checks per object and identifier guidance | web_fetch |
| 7 | https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html | Per-request authorization and ID lookup risks | web_fetch |
| 8 | https://cheatsheetseries.owasp.org/cheatsheets/Multi_Tenant_Security_Cheat_Sheet.html | Tenant context, cache/session, file/blob isolation | web_fetch |
| 9 | https://cheatsheetseries.owasp.org/cheatsheets/GraphQL_Cheat_Sheet.html | GraphQL edge/node and resolver authorization | web_fetch |
| 10 | https://cheatsheetseries.owasp.org/cheatsheets/WebSocket_Security_Cheat_Sheet.html | WebSocket per-message authorization | web_fetch |
| 11 | https://portswigger.net/web-security/web-cache-deception | Cache/CDN private response gadgets | web_fetch |
| 12 | https://cheatsheetseries.owasp.org/cheatsheets/REST_Security_Cheat_Sheet.html | Method auth and out-of-order workflow checks | web_fetch |
| 13 | https://cheatsheetseries.owasp.org/cheatsheets/Business_Logic_Security_Cheat_Sheet.html | Business-layer authorization and workflow state | web_fetch |
| 14 | https://portswigger.net/web-security/race-conditions | Race and hidden multi-step sequence amplifiers | web_fetch |
| 15 | https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html | File storage and retrieval authorization context | web_fetch |

