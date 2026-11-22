# Building Tool-Protocol Servers in Go

Building tool-protocol servers in Go against MCP 2026-07-28 — the official SDK, stateless transports, tool registration, per-session capability without protocol sessions, implementation notes, and the telemetry/testing seams.

Tool-protocol (MCP) servers are the standard shape for giving agents safe,
typed access to domain capability, and Go suits them well: small statically
linked binaries, strong typing on tool schemas, and mature context
propagation. The protocol-level facts this file builds on — the stateless
2026-07-28 revision and its deprecations — are in the protocols reference.

## The SDK

Use the official Go SDK, `github.com/modelcontextprotocol/go-sdk` — one of the
four Tier 1 MCP SDKs. Its `mcp` package carries client and server APIs,
`jsonrpc` is for custom transports, and `auth` / `oauthex` carry the OAuth
resource-server primitives (including Protected Resource Metadata). As of
2026-11-22:

- **v1.7.0** (2026-07-28) added protocol revision 2026-07-28 — the wire layer
  was largely rewritten around the stateless model — while still speaking
  2025-11-25 and earlier revisions to older peers;
- **v1.8.0** (2026-09-14) is the latest stable release; v1.9.0 is in
  pre-release, with no newer protocol revision.

Pin the SDK version where dependencies are pinned and read its release notes
on every bump: the v1.7 line shipped temporary `MCPGODEBUG` escape hatches
whose removal is scheduled in later minors. Hand-rolling JSON-RPC framing for
a server the SDK can serve is the same mistake as hand-writing the protocol
schema.

## Transports

- **stdio** — the server is spawned as a child process of the agent host and
  speaks JSON-RPC over stdin/stdout. Co-located, zero network surface, the
  default for local tool servers. stdout carries only protocol frames —
  diagnostics go to stderr, or they corrupt the stream. Credentials come from
  the launch environment, not an OAuth flow.
- **Streamable HTTP** — the server is a network service; each request is an
  independent POST whose response may stream request-scoped progress. The
  right shape when several agents or callers share one server, or when the
  tools live behind cluster networking. In the Go SDK the Streamable HTTP
  handler accepts 2026-07-28 requests only with
  `StreamableHTTPOptions.Stateless` set; without it, clients negotiate down to
  2025-11-25. New servers set it.
- **HTTP+SSE** (the two-endpoint transport) is deprecated — never build on it.

Pick per deployment, not per preference: stdio for per-session lifetime and
tight coupling, Streamable HTTP for shared servers and independent scaling.
Because requests are self-describing, a stateless Streamable HTTP server
scales horizontally behind an ordinary load balancer with no session
affinity.

## Tool registration

Each tool is registered with a descriptor: a stable name (namespaced by the
server's domain — `domain.action`), a description written for model
consumption (it is prompt surface; precision here is tool-selection
accuracy), and a JSON Schema for input (and, where results are structured,
output) validation; 2026-07-28 admits any JSON Schema 2020-12 keyword. In the
Go SDK the generic `mcp.AddTool` takes a typed input and output struct and
derives the schemas from them. Handlers stay thin: parse the
schema-validated input, call the domain logic, return typed results with
explicit error semantics.

Return `tools/list` in a deterministic order and set honest `ttlMs` /
`cacheScope` hints: clients cache the list, and a stable order keeps model
prompt caches warm.

## Per-session capability without protocol sessions

Skill bundles mount tool sets on demand (see the skills reference), but
2026-07-28 has no protocol-level session and list results do not vary per
connection. Mounting therefore lives on the **host side**:

- The agent runtime decides which tools from which servers enter a given
  model session; a server exposes its full, stable tool set and is unaware of
  skills. Split servers along skill boundaries when a skill's tools should be
  separately deployable or separately authorized.
- Authorization narrows what a caller may *invoke*, not what `tools/list`
  shows: a handler rejects an out-of-scope call with an authorization error.
- A tool set that genuinely changes at runtime (a plugin installed on the
  server) is announced through a `toolsListChanged` notification on
  `subscriptions/listen`, for every subscriber — not as a per-caller list.
- State a workflow needs across calls is an explicit **server-minted handle**
  returned by one tool and passed back as an ordinary argument to the next —
  keyed, expiring, and discardable. Nothing is keyed on a connection.

## Implementation notes

- Schema-first: the JSON Schema is the contract; derive or validate it from
  the tool's typed input rather than hand-syncing two representations.
- Propagate `context.Context` through every handler; cancellation and
  deadlines arrive per request, and a broken response stream means the client
  re-issues the request — make tools idempotent or deduplicate on a
  caller-supplied key.
- Long-running tools emit progress on the request's response stream instead
  of blocking silently; work that outlives a request belongs to the tasks
  extension, not to an open connection.
- Mid-call confirmation or missing parameters use Multi Round-Trip Requests:
  return an `input_required` result and finish when the client retries with
  the answers. Do not adopt Roots, Sampling, or protocol Logging in new
  servers — take paths as parameters, call model providers directly, and log
  to stderr or OpenTelemetry.

## Seams owned elsewhere

Three concerns hook in here but are governed elsewhere, named in prose only:
**authorization** — a networked server is an OAuth 2.1 resource server with
audience-bound tokens; the requirements are the authorization reference's.
**Telemetry** — a tool invocation should emit an AI span with tool-selection
and result attributes, continuing the trace context the client propagates in
`_meta` (`traceparent`, `tracestate`); the span semantics are this profile's
(the `telemetry` package), while transport-level interceptor wiring is the Go
services profile's concern. **Verification** — deterministic tool behavior
is verified by external black-box test tools against the built server
binary, not by tests inside the source package; the general testing split is
the core profile's.

## Related

- <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::orchestration@agentic_development/reference/protocols_mcp_a2a.md" />
- <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::orchestration@agentic_development/reference/mcp_authorization.md" />
- <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::orchestration@agentic_development/reference/skills.md" />
- <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::telemetry@agentic_development/reference/otel_genai.md" />

## Sources

- https://github.com/modelcontextprotocol/go-sdk/releases — v1.7.0 (2026-07-28, revision 2026-07-28, `StreamableHTTPOptions.Stateless`), v1.8.0 (2026-09-14, latest stable), v1.9.0-pre.1, `MCPGODEBUG` removal schedule, accessed 2026-11-22
- https://github.com/modelcontextprotocol/go-sdk — package layout (`mcp`, `jsonrpc`, `auth`, `oauthex`), `mcp.AddTool`, supported revisions, accessed 2026-11-22
- https://pkg.go.dev/github.com/modelcontextprotocol/go-sdk/mcp — `AddTool[In, Out]` infers input and output schemas from the handler's type parameters and validates input, accessed 2026-11-22
- https://modelcontextprotocol.io/specification/2026-07-28/changelog — stateless core, handles instead of sessions, deterministic `tools/list`, cache hints, MRTR, `_meta` trace context (SEP-414), deprecations, accessed 2026-11-22
- https://blog.modelcontextprotocol.io/posts/2026-07-28/ — Tier 1 SDKs (TypeScript, Python, Go, C#) and server migration themes, accessed 2026-11-22
