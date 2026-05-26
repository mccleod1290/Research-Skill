# Curated Sources: access-control IDOR BOLA BFLA business-logic gadgets

## Top sources (ranked by rarity + signal)

1. **[RustFS GHSA-pfcq-4gjr-6gjm: Missing admin authorization on notification target endpoints](https://github.com/rustfs/rustfs/security/advisories/GHSA-pfcq-4gjr-6gjm)** — Event/webhook configuration is a high-impact gadget when low-privileged users can alter delivery targets or suppress audit visibility.
2. **[axios-cache-interceptor GHSA-x4m5-4cw8-vc44: ignored Vary Authorization](https://github.com/arthurfiorette/axios-cache-interceptor/security/advisories/GHSA-x4m5-4cw8-vc44)** — Cache identity missing auth context can convert a valid request into cross-user data disclosure.
3. **[Common Authorization Failures in API-Heavy Web Applications](https://packetwanderer.com/posts/api-authorization-failures/)** — The best general source for authorization drift across async jobs, exports, bulk actions, workflows, and convenience endpoints.
4. **[WorkOS: How to design an RBAC model for multi-tenant SaaS](https://workos.com/blog/how-to-design-multi-tenant-rbac-saas)** — Permission caching needs tenant-aware keys or roles can bleed across organizations.
5. **[Stingrai: GraphQL API Vulnerabilities and Common Attacks](https://www.stingrai.io/blog/graphql-api-vulnerabilities-and-common-attacks)** — GraphQL turns access control into per-resolver and per-field enforcement, with real BOLA/BFLA report anchors.
6. **[Bug Bounty Playbook: GraphQL](https://bugbounty.info/Attack-Surface/API/GraphQL)** — Strong hunter source for nested resolver authorization, schema reconstruction, alias batching, and mutation gaps.
7. **[Apiiro: Why DAST Tools Miss Real IDOR Vulnerabilities](https://apiiro.com/blog/why-dast-tools-miss-real-idor-vulnerabilities-and-how-ai-helps/)** — Explains why secondary endpoints like exports, invoice downloads, and bulk operations escape signature-based testing.
8. **[Agnite Studio: IDOR Testing for SaaS](https://agnitestudio.com/saas-security-audit/idor-testing/)** — Good evidence model for replaying the same object under user, tenant, and role mismatch.
9. **[Figma SCIM API Reference](https://developers.figma.com/docs/rest-api/scim/)** — SCIM tenant URLs and admin-generated bearer tokens make lifecycle provisioning a non-obvious access-control surface.
10. **[HID SCIM Search API](https://docs.hidglobal.com/dev/auth-service/api/searching-with-the-scim-api.htm)** — SCIM search can cover users, devices, credentials, audit records, and bulk-import correlation IDs inside a tenant.
11. **[Azure Event Grid: Secure webhook delivery with Microsoft Entra ID](https://learn.microsoft.com/en-us/azure/event-grid/secure-webhook-delivery)** — Event delivery authentication must be validated by webhook receivers, especially across tenants.
12. **[Semgrep: Can LLMs Detect IDORs?](https://semgrep.dev/blog/2025/can-llms-detect-idors-understanding-the-boundaries-of-ai-reasoning/)** — Useful for mapping code-level IDOR difficulty: missing checks, local checks, spread-out RBAC, and implicit middleware.
13. **[GitHub Docs: Scopes for OAuth apps](https://docs.github.com/en/apps/oauth-apps/building-oauth-apps/scopes-for-oauth-apps)** — OAuth app scopes are durable-access gadgets for hooks, org membership, audit logs, packages, workflows, and projects.
14. **[RFC 9700: Best Current Practice for OAuth 2.0 Security](https://www.rfc-editor.org/rfc/rfc9700.html)** — Current OAuth security baseline for redirect, token replay, issuer/mix-up, and access-token privilege restriction.
15. **[OWASP Multi-Tenant Security Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Multi_Tenant_Security_Cheat_Sheet.html)** — Canonical baseline for tenant context, composite keys, cache isolation, queue/storage poisoning, and tenant audit gaps.
16. **[Bugcrowd: How to find bugs on a hardened target using gadgets](https://www.bugcrowd.com/blog/how-to-find-bugs-on-a-hardened-target-using-gadgets/)** — Seed framing for chaining low-severity quirks into higher-impact findings.

## Novel patterns spotted

- Event-sink takeover: webhook, notification, audit, and integration targets are not just outputs; they are durable exfiltration gadgets when CRUD permissions are weaker than the event sources they subscribe to.
- Async artifact split-brain: authorization may be correct at job creation but weak at job execution, status polling, result retrieval, or generated-file download.
- Cache-key authorization drift: cache keys that omit tenant, user, role, scope, or `Authorization` dimensions can turn correct backend auth into cross-user disclosure.
- Nested resolver escape paths: GraphQL top-level queries may be protected while nested fields, mutations, aliases, and subscription/event dispatch paths reuse weaker checks.
- SCIM lifecycle overreach: provisioning APIs touch users, groups, devices, credentials, audit records, and bulk-import status; tenant URL and token binding deserve the same IDOR treatment as app APIs.
- Delegated-access persistence: OAuth scopes, refresh tokens, app installs, webhooks, and integration tokens can preserve or widen access after a small access-control primitive.
- Evidence-side gadgets: audit logs, notification previews, report metadata, and export manifests can leak object IDs, state transitions, tenant names, email addresses, or correlation IDs even when primary objects are protected.

## Coverage gaps

- Public sources for websocket/channel authorization were thinner than expected; GraphQL subscriptions and event streams partially cover the pattern.
- Mobile/legacy endpoint examples were mostly generic, not high-signal enough for this v3 cut.
- File preview/OCR/transcoding had strong conceptual signal but fewer rare public sources than webhook/cache/SCIM/GraphQL.
