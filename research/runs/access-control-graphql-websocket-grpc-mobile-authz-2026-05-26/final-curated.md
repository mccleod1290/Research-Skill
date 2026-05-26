# Curated Sources: access-control bug bounty leads on hardened GraphQL/WebSocket/gRPC/mobile authorization surfaces

## Top sources

1. ★5 [API Platform GHSA-cg3c-245w-728m: GraphQL query operations security can be bypassed](https://github.com/api-platform/core/security/advisories/GHSA-cg3c-245w-728m) — Concrete Relay `node(id:)` operation-security bypass.
2. ★5 [gRPC-Gateway: Customizing your gateway](https://grpc-ecosystem.github.io/grpc-gateway/docs/mapping/customizing_your_gateway/) — Header-to-metadata and method-override access-control confusion source.
3. ★4 [GitLab GraphQL Authorization](https://docs.gitlab.com/development/graphql_guide/authorization/) — Global ID loading and error-oracle guidance.
4. ★4 [Shopify: Global IDs in Shopify APIs](https://shopify.dev/docs/api/usage/gids) — Parameterized global IDs with child/parent components.
5. ★4 [Dgraph GraphQL Subscriptions](https://docs.dgraph.io/graphql/subscriptions/) — WebSocket `connectionParams` and subscription auth rules.
6. ★4 [AWS Amplify: Customize authorization rules](https://docs.amplify.aws/gen1/javascript/build-a-backend/graphqlapi/customize-authorization-rules/) — Owner, group, custom claim, and real-time operation auth.
7. ★4 [AWS AppSync: Authorization and authentication](https://docs.aws.amazon.com/appsync/latest/devguide/security-authz.html) — Mixed AppSync auth modes at schema field/type level.
8. ★4 [Hasura JWT Auth](https://hasura.io/docs/2.0/auth/authentication/jwt/) — Role and session-variable claims used directly in GraphQL authorization.
9. ★4 [Hasura Row Permissions](https://hasura.io/docs/2.0/auth/authorization/permissions/row-level-permissions/) — Row-level filters across owners, relationships, and remote relationships.
10. ★4 [Pusher Channels: Auth signatures](https://pusher.com/docs/channels/library_auth_reference/auth-signatures/) — Exact `socket_id` and `channel_name` channel authorization parameters.
11. ★4 [Socket.IO Rooms](https://socket.io/docs/v4/rooms/) — Room naming patterns such as per-user and per-project subscriptions.
12. ★4 [Socket.IO Middlewares](https://socket.io/docs/v4/middlewares/) — One-time connection middleware and `handshake.auth` credentials.
13. ★4 [Rails ActionCable::Channel::Base](https://railsdoc.github.io/8.0/classes/ActionCable/Channel/Base.html) — Subscription params, long-lived channel objects, and callable channel actions.
14. ★4 [Envoy gRPC-JSON Transcoder](https://www.envoyproxy.io/docs/envoy/latest/configuration/http/http_filters/grpc_json_transcoder_filter) — REST-to-gRPC route remapping and original-path/original-method headers.
15. ★4 [Firebase Security Rules basics](https://firebase.google.com/docs/rules/basics) — Mobile direct-object APIs with `request.auth.uid` and owner fields.

## Novel patterns spotted

- Alternate GraphQL object loaders: direct field auth may be correct while `node(id:)`, Global ID loaders, REST-to-GraphQL ID pivots, or mutations accepting global `id` inputs miss the same object policy.
- Stateful real-time authorization: WebSocket subscriptions authenticate once, then keep `room`, `channel_name`, `connectionParams`, or owner context alive while memberships and JWT claims change.
- Metadata-as-tenant-context: gRPC and gateway layers often translate HTTP headers into metadata; tenant/user values can drift from the JWT subject unless the backend binds them.
- Mixed-mode schema auth: AppSync/Amplify/Hasura permissions split by role, provider, operation, field, and real-time mode, so `get`, `list`, `sync`, `listen`, `search`, and subscription paths need separate checks.
- Mobile direct datastore rules: Firebase-style backends expose object paths and owner fields directly to clients, making ownership-change fields and query shape part of the access-control boundary.
