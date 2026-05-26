# V2 Scout Brief: Sources and Facts

**source_coverage:** complete for requested artifact.

## Facts

- Bugcrowd's gadget article frames a gadget as a small, low-impact bug or odd behavior that becomes high impact when chained. The article stresses note taking, understanding the target, and asking what one small behavior can feed into next. Source: https://www.bugcrowd.com/blog/how-to-find-bugs-on-a-hardened-target-using-gadgets/
- OWASP API Top 10 2023 splits the access-control space into object-level authorization, property-level authorization, function-level authorization, sensitive business flows, and inventory/version drift. Source: https://owasp.org/API-Security/editions/2023/en/0x11-t10/
- OWASP API1 says endpoints that receive an object ID and act on it should check that the logged-in user has permission for that object and action. Source: https://owasp.org/API-Security/editions/2023/en/0xa1-broken-object-level-authorization/
- OWASP API3 covers sensitive properties returned or modified without field-level permission, including mass assignment style writes. Source: https://owasp.org/API-Security/editions/2023/en/0xa3-broken-object-property-level-authorization/
- OWASP API5 covers users reaching functions they should not access, including admin endpoints and method changes. Source: https://owasp.org/API-Security/editions/2023/en/0xa5-broken-function-level-authorization/
- OWASP IDOR prevention says access control must be checked for each object access; complex IDs help only as defense in depth. Source: https://cheatsheetseries.owasp.org/cheatsheets/Insecure_Direct_Object_Reference_Prevention_Cheat_Sheet.html
- OWASP Multi-Tenant Security calls out cross-tenant data leakage, tenant context injection, shared resource poisoning, cache/session isolation, and file/blob isolation. Source: https://cheatsheetseries.owasp.org/cheatsheets/Multi_Tenant_Security_Cheat_Sheet.html
- OWASP GraphQL says to authorize both edges and nodes, and to validate read and mutation access per resolver. Source: https://cheatsheetseries.owasp.org/cheatsheets/GraphQL_Cheat_Sheet.html
- OWASP WebSocket Security says not to treat a WebSocket connection as unlimited access, and to check authorization for each action. Source: https://cheatsheetseries.owasp.org/cheatsheets/WebSocket_Security_Cheat_Sheet.html
- PortSwigger Web Cache Deception explains that discrepancies between cache and origin parsing can store sensitive dynamic responses under static-looking cache rules. Source: https://portswigger.net/web-security/web-cache-deception
- OWASP REST Security says non-public REST endpoints must perform access control at each endpoint, method authorization must include the resource/action/record, and workflow state must be validated server side. Source: https://cheatsheetseries.owasp.org/cheatsheets/REST_Security_Cheat_Sheet.html
- OWASP Business Logic Security says server-side workflow state, server-derived security values, and atomic operations matter because many logic bugs are authorization bugs in disguise. Source: https://cheatsheetseries.owasp.org/cheatsheets/Business_Logic_Security_Cheat_Sheet.html

## Open Questions

- No target-specific implementation is available, so endpoint names in the final artifact must stay generic.
- Public bounty writeups can sharpen examples, but canonical sources already cover the required WHY-level model. The v3 source hunt records optional follow-up sources separately.

