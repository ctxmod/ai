# MCP Authorization

Authorization for networked tool-protocol servers under MCP 2026-07-28 — the server as an OAuth 2.1 resource server, audience-bound tokens, issuer validation, and client registration by metadata document.

MCP authorization is optional and transport-level. It applies to HTTP-based
transports only: a **stdio** server does not run the OAuth flow and takes its
credentials from the environment its host launches it with. Everything below
concerns Streamable HTTP servers and the clients that call them.

## Roles

- The **MCP server** is an OAuth 2.1 **resource server**: it validates access
  tokens and never issues them.
- The **MCP client** is an OAuth 2.1 client acting on behalf of a resource
  owner (or on its own behalf, with `client_credentials`).
- The **authorization server** issues tokens; it may be co-hosted with the
  MCP server or be the organization's existing identity provider.

Keep the roles apart even when one deployment hosts two of them: a server that
mints its own tokens ad hoc has re-invented an authorization server without
its security properties.

## Server obligations

- **Publish Protected Resource Metadata** (RFC 9728) and point to it from the
  `WWW-Authenticate` header of every 401, so clients discover the authorization
  server instead of being configured with it.
- **Validate the audience.** Accept only tokens issued for this server's
  canonical URI (RFC 8707 resource indicators); invalid or expired tokens get
  401, insufficient scope gets 403 with `error="insufficient_scope"` and the
  full set of scopes the operation needs in one challenge.
- **Never pass tokens through.** A server that calls an upstream API obtains a
  token for that upstream itself; it must not accept, or forward, a token
  minted for anyone else. Token passthrough is the confused-deputy hole the
  audience rule closes.
- **Advertise minimal scopes.** `scopes_supported` is the minimal set for basic
  use; further scopes are requested by step-up authorization when an operation
  needs them, not up front.
- Tokens travel only in the `Authorization: Bearer` header of every request,
  never in a query string.

## Client obligations

- **Discover, then register.** Read the resource metadata, then the
  authorization server's metadata (RFC 8414 or OpenID Connect Discovery — a
  client must support both). Obtain a client ID by **Client ID Metadata
  Documents** (an HTTPS URL as `client_id`), by pre-registration, or — only for
  authorization servers that support nothing better — by Dynamic Client
  Registration, which 2026-07-28 deprecates.
- **PKCE and resource indicators** on every authorization and token request;
  the `resource` parameter names the MCP server's canonical URI whether or not
  the authorization server supports it.
- **Validate `iss`** (RFC 9207): record the expected issuer from validated
  metadata alongside the PKCE verifier, and compare a present `iss` in the
  authorization response — including error responses — by exact string
  comparison before redeeming the code. Reject a missing `iss` when the server
  advertises `authorization_response_iss_parameter_supported`. A future
  revision is expected to make `iss` mandatory; emit and validate it now.
- **Bind credentials to their issuer.** Persisted client credentials are keyed
  by issuer identifier and never reused with a different authorization server;
  a changed authorization server means re-registration.
- **Step up by union.** On a scope challenge, request the union of previously
  granted and newly challenged scopes, with a retry limit.

## Where this lives in a codebase

Token validation is middleware in front of the MCP handler, not logic inside
tool handlers: a handler receives an authenticated principal and its scopes,
and authorizes the specific operation against them. Credential storage,
workload identity, and secret delivery follow the repository's IAM
conventions, owned by the delivery and IAM profiles.

## Related

- <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::orchestration@agentic_development/reference/protocols_mcp_a2a.md" />
- <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::orchestration@agentic_development/reference/mcp_servers_go.md" />

## Sources

- https://modelcontextprotocol.io/specification/2026-07-28/basic/authorization — normative authorization flow: roles, RFC 9728 discovery, RFC 8707 audience binding, RFC 9207 `iss` validation, scope challenges, stdio exemption, revision 2026-07-28, accessed 2026-11-22
- https://modelcontextprotocol.io/specification/2026-07-28/changelog — SEP-2468 (`iss`), SEP-2352 (issuer-bound credentials), SEP-837 (`application_type`), deprecation of Dynamic Client Registration in favor of Client ID Metadata Documents, accessed 2026-11-22
- https://datatracker.ietf.org/doc/html/rfc9207 — OAuth 2.0 Authorization Server Issuer Identification, accessed 2026-11-22
