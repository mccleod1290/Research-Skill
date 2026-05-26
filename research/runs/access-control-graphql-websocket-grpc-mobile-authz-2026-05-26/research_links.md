# Research: access-control bug bounty leads on hardened GraphQL/WebSocket/gRPC/mobile authorization surfaces

## Seeds

## Round 1
- ★3 [GraphQL Global Object Identification](https://graphql.org/learn/global-object-identification/) — Canonical source for `Node`, `node(id: ID!)`, and refetch-by-ID behavior that creates alternate object access paths.
- ★3 [GitHub Docs: Using global node IDs](https://docs.github.com/en/graphql/guides/using-global-node-ids?apiVersion=2022-11-28) — Shows REST `node_id` to GraphQL `node(id:)` pivot and `__typename` probing for direct node lookups.
- ★4 [Shopify: Global IDs in Shopify APIs](https://shopify.dev/docs/api/usage/gids) — High-signal because it documents parameterized global IDs such as child IDs with parent query parameters, a rarer parent/child authorization mismatch surface.
- ★5 [API Platform GHSA-cg3c-245w-728m: GraphQL query operations security can be bypassed](https://github.com/api-platform/core/security/advisories/GHSA-cg3c-245w-728m) — Concrete 2025 advisory where Relay `node(id:)` bypassed configured operation security. [Justification: primary GitHub advisory, specific to GraphQL Node auth bypass, not a generic IDOR post.]

## Round 2
- ★4 [GitLab GraphQL Authorization](https://docs.gitlab.com/development/graphql_guide/authorization/) — Rarely cited developer guidance warning that GraphQL-Ruby `loads:` can leak whether Global IDs exist versus are unauthorized.
- ★4 [Dgraph GraphQL Subscriptions](https://docs.dgraph.io/graphql/subscriptions/) — Subscription-specific auth source showing `connectionParams`, `X-Dgraph-AuthToken`, `@withSubscription`, and `@auth` applied over WebSocket state.
- ★4 [Hasura Row Permissions](https://hasura.io/docs/2.0/auth/authorization/permissions/row-level-permissions/) — Precise row-filter examples using `X-Hasura-User-Id`, nested relationships, and remote source relationships.
- ★4 [Hasura JWT Auth](https://hasura.io/docs/2.0/auth/authentication/jwt/) — Documents `X-Hasura-Role`, `x-hasura-allowed-roles`, `x-hasura-default-role`, and arbitrary `x-hasura-*` claims used in permissions.

## Round 3
- ★4 [AWS Amplify: Customize authorization rules](https://docs.amplify.aws/gen1/javascript/build-a-backend/graphqlapi/customize-authorization-rules/) — High-signal AppSync/Amplify source for `ownerField`, `groupsField`, `identityClaim`, `groupClaim`, and operation splits including `listen` and `sync`.
- ★4 [AWS AppSync: Authorization and authentication](https://docs.aws.amazon.com/appsync/latest/devguide/security-authz.html) — Documents field/type-level auth modes (`@aws_api_key`, `@aws_iam`, `@aws_oidc`, `@aws_cognito_user_pools`, `@aws_lambda`) and mixed-mode schema behavior.
- ★4 [Pusher Channels: Auth signatures](https://pusher.com/docs/channels/library_auth_reference/auth-signatures/) — Gives exact private/presence channel auth signing inputs, especially `socket_id` and `channel_name`.
- ★4 [Pusher Channels: Channel types and naming](https://pusher.com/docs/channels/using_channels/channels/) — Useful for hunting channel-name authorization boundaries across `private-`, `presence-`, and `private-encrypted-` namespaces.
- ★4 [Socket.IO Middlewares](https://socket.io/docs/v4/middlewares/) — Documents `handshake.auth.token` and one-time-per-connection middleware behavior, a common stale authz surface.
- ★4 [Socket.IO Rooms](https://socket.io/docs/v4/rooms/) — Exact room patterns such as user rooms and `project:{id}` rooms that expose object-level authorization joins.
- ★4 [Rails ActionCable::Channel::Base](https://railsdoc.github.io/8.0/classes/ActionCable/Channel/Base.html) — Shows `params[:room_number]`, long-lived channel state, public RPC-like actions, and `reject`-based subscription authorization.

## Round 4
- ★5 [gRPC-Gateway: Customizing your gateway](https://grpc-ecosystem.github.io/grpc-gateway/docs/mapping/customizing_your_gateway/) — Rare source tying header-to-metadata forwarding, `authorization`, custom `X-User-Id`, and `X-HTTP-Method-Override` to access-control confusion. [Justification: gateway-specific authz edge case, including method-override bypass notes, much rarer than generic gRPC auth docs.]
- ★4 [gRPC Metadata](https://grpc.io/docs/guides/metadata/) — Precise metadata model: HTTP/2 headers, case-insensitive keys, `authorization`, and custom headers for per-RPC authz context.
- ★4 [gRPC Authentication](https://grpc.io/docs/guides/auth/) — Shows per-call credentials and custom metadata header injection, useful for hunting authz drift between channel credentials and per-method checks.
- ★4 [Envoy gRPC-JSON Transcoder](https://www.envoyproxy.io/docs/envoy/latest/configuration/http/http_filters/grpc_json_transcoder_filter) — Documents route transformation to `/<package>.<service>/<method>` and forwarded `x-envoy-original-path` / `x-envoy-original-method` headers.
- ★4 [Firebase Realtime Database Security Rules](https://firebase.google.com/docs/database/security) — Mobile/backend-as-a-service source showing server-enforced `.read`/`.write` rules and owner paths like `/users/<uid>`.
- ★4 [Firebase Security Rules basics](https://firebase.google.com/docs/rules/basics) — Firestore/Storage examples for `request.auth.uid`, `request.resource.data.author_uid`, `resource.data.author_uid`, and ownership-change prevention.
