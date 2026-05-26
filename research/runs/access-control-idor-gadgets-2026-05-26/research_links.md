# Research Links: Access-Control and IDOR Gadgets

## Seeds

- [Bugcrowd - How to find bugs on a hardened target using gadgets](https://www.bugcrowd.com/blog/how-to-find-bugs-on-a-hardened-target-using-gadgets/) - seed framing for saving low-impact quirks and chaining them later.

## Round 1 - Canonical boundaries

- [r5] [OWASP API Security Top 10 2023](https://owasp.org/API-Security/editions/2023/en/0x11-t10/) - splits API authorization into object, property, function, flow, and inventory classes.
- [r5] [OWASP API1 Broken Object Level Authorization](https://owasp.org/API-Security/editions/2023/en/0xa1-broken-object-level-authorization/) - base model for IDOR/BOLA gadgets.
- [r5] [OWASP API3 Broken Object Property Level Authorization](https://owasp.org/API-Security/editions/2023/en/0xa3-broken-object-property-level-authorization/) - source for hidden field reads and mass-assignment writes.
- [r5] [OWASP API5 Broken Function Level Authorization](https://owasp.org/API-Security/editions/2023/en/0xa5-broken-function-level-authorization/) - source for admin/helper/function gadgets.
- [r5] [PortSwigger Access Control](https://portswigger.net/web-security/access-control) - clear horizontal, vertical, and context-dependent model.
- [r5] [PortSwigger IDOR](https://portswigger.net/web-security/access-control/idor) - practical object and static-file IDOR frame.

## Round 2 - Channels that drift from REST

- [r4] [OWASP GraphQL Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/GraphQL_Cheat_Sheet.html) - edge/node resolver authorization and property-level access.
- [r4] [OWASP WebSocket Security Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/WebSocket_Security_Cheat_Sheet.html) - message-level authorization for long-lived channels.
- [r4] [PortSwigger WebSockets](https://portswigger.net/web-security/websockets) - WebSocket vulnerabilities mirror normal HTTP bugs and handshake trust mistakes matter.
- [r4] [PortSwigger Web Cache Deception](https://portswigger.net/web-security/web-cache-deception) - cache/origin parsing differences can turn private dynamic data into cacheable artifacts.
- [r4] [OWASP REST Security Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/REST_Security_Cheat_Sheet.html) - endpoint-level access control, method authorization, and out-of-order API execution.

## Round 3 - Tenant, workflow, and storage amplifiers

- [r4] [OWASP Multi-Tenant Security Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Multi_Tenant_Security_Cheat_Sheet.html) - tenant context, cache/session isolation, file/blob isolation, and cross-tenant risks.
- [r4] [OWASP Business Logic Security Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Business_Logic_Security_Cheat_Sheet.html) - workflows, state machines, server-derived security values, and business-layer authorization.
- [r4] [OWASP WSTG - Circumvention of Work Flows](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/10-Business_Logic_Testing/06-Testing_for_the_Circumvention_of_Work_Flows) - manual misuse-case thinking for workflow bypasses.
- [r4] [PortSwigger Race Conditions](https://portswigger.net/web-security/race-conditions) - race windows and hidden multi-step sequences as logic amplifiers.
- [r4] [OWASP File Upload Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html) - file storage, download handling, and authorization around public retrieval.

## Novel patterns spotted

- Channel drift is the richest gadget source: REST may be fixed while exports, async jobs, GraphQL resolvers, WebSockets, mobile APIs, or CDN paths still trust stale context.
- Authorization needs to bind more than object ownership: field, parent, child, tenant, workflow state, channel, and token scope each create separate break points.
- Low-impact reads become higher impact when paired with blast-radius features: exports, reports, batch actions, search indexes, webhooks, and audit feeds.
- Persistence gadgets change severity fastest: invites, ownership transfer, API tokens, OAuth apps, share links, and integration mappings survive the first weak request.

