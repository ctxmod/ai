# Building Tool-Protocol Servers in Go

Building tool-protocol servers in Go — transports, tool registration, dynamic mounting, implementation notes, and the telemetry/testing seams.

Tool-protocol (MCP) servers are the standard shape for giving agents safe,
typed access to domain capability, and Go suits them well: small statically
linked binaries, strong typing on tool schemas, and mature context
propagation.

## Transports

- **stdio** — the server is spawned as a child process of the agent host and
  speaks JSON-RPC over stdin/stdout. Co-located, zero network surface, the
  default for local tool servers. stdout carries only protocol frames —
  diagnostics go to stderr, or they corrupt the stream.
- **Streamable HTTP / SSE** — the server is a network service; sessions
  carry a stream for server-initiated updates. The right shape when several
  agents or callers share one server, or when the tools live behind cluster
  networking.

Pick per deployment, not per preference: stdio for per-session lifetime and
tight coupling, HTTP for shared servers and independent scaling.

## Tool registration

Each tool is registered with a descriptor: a stable name (namespaced by the
server's domain — `domain.action`), a description written for model
consumption (it is prompt surface; precision here is tool-selection
accuracy), and a JSON Schema for input validation. Handlers stay thin: parse
the schema-validated input, call the domain logic, return typed results with
explicit error semantics.

## Dynamic mounting

Skill bundles mount tool sets at session time (see the skills reference), so
servers must support session-scoped registration: a mount call attaches a
tool set to a session and a corresponding detach removes it, with discovery
(`tools/list`) reflecting the session's current set. Keep server state
effectively stateless — per-session state is explicit, keyed by session id,
and discardable; nothing survives a session that the session did not ask to
persist.

## Implementation notes

- Schema-first: the JSON Schema is the contract; generate or validate it
  against the tool's typed input rather than hand-syncing two
  representations.
- Propagate `context.Context` through every handler; cancellation and
  deadlines arrive per request.
- Long-running tools emit progress on the session stream instead of
  blocking silently.

## Seams owned elsewhere

Two concerns hook in here but are governed elsewhere, named in prose only:
**telemetry** — a tool invocation should emit an AI-span with tool-selection
and result attributes; the span semantics are this profile's
(the `telemetry` package), while transport-level interceptor wiring is the Go
services profile's concern. **Verification** — deterministic tool behavior
is verified by external black-box test tools against the built server
binary, not by tests inside the source package; the general testing split is
the core profile's.

## Related

- <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::orchestration@agentic_development/reference/protocols_mcp_a2a.md" />
- <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::orchestration@agentic_development/reference/skills.md" />
- <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::telemetry@agentic_development/reference/openinference.md" />
