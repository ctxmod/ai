# Agent Protocols: MCP and A2A

The protocol division of labor — MCP vertically for agent-to-tools, A2A horizontally for client-to-agent — the current revision of each, and vendored-schema governance for both.

The industry has consolidated on two canonical open protocols covering the two
boundaries in an agent stack. Both are hosted by the Linux Foundation's
Agentic AI Foundation (AAIF), alongside AGENTS.md, goose, and agentgateway. The
architectural mistake to avoid is inventing either boundary ad hoc, or using
one protocol for both jobs.

## The division of labor

- **MCP (Model Context Protocol) is vertical: agent ↔ tools.** It defines
  how an agent discovers and invokes tools (`tools/list`, `tools/call`), reads
  contextual resources, and fetches prompts. Tools are primitives with
  well-defined inputs and outputs; the agent *uses* them.
- **A2A (Agent2Agent) is horizontal: client ↔ agent, and agent ↔ agent.** It
  defines capability discovery, a standardized task lifecycle, and streamed
  artifacts between a caller and an **opaque** agent — one that reasons,
  plans, and keeps state the caller never sees. The other agent is a peer it
  *partners* with, often owned by another team or organization.

The test when a boundary is ambiguous: if the callee is a deterministic
capability with a schema, it is an MCP tool; if it is an autonomous agent
whose internals are not the caller's business and whose work may span many
turns, it is an A2A agent. Wrapping a whole agent as one MCP tool hides its
task lifecycle; exposing a plain function as an A2A agent invents one.

## The three-tier mapping

A typical stack routes each boundary through its own protocol:

```
client (UI / service)
   │  A2A — capability discovery, task lifecycle, streamed artifacts
   ▼
agent runtime ──A2A──▶ peer agents (other teams, partners)
   │  MCP — tool discovery, invocation, resources, prompts
   ▼
tool servers
```

## MCP: the current revision

The current MCP revision is **2026-07-28** (released 2026-07-28; revisions are
date-stamped by their last backwards-incompatible change). It is a breaking
revision built around a **stateless core**:

- **No handshake, no protocol sessions.** The `initialize` /
  `notifications/initialized` handshake and the `Mcp-Session-Id` header are
  gone. Every request carries its protocol version and client capabilities in
  `_meta` (`io.modelcontextprotocol/protocolVersion`,
  `io.modelcontextprotocol/clientCapabilities`); a version the server does not
  speak returns `UnsupportedProtocolVersionError` listing the ones it does.
- **`server/discover`** — a mandatory RPC returning the server's supported
  versions, capabilities, and identity; clients may call it up front or simply
  send requests and handle a version error.
- **List results are not per-connection.** `tools/list`, `prompts/list`, and
  `resources/list` return the same answer to every caller, in a deterministic
  order, with `ttlMs` / `cacheScope` cache hints. State a server needs across
  calls lives in explicit, server-minted handles passed as ordinary tool
  arguments.
- **Multi Round-Trip Requests (MRTR)** replace server-initiated requests: a
  server that needs input (an elicitation, a confirmation) returns a result
  with `resultType: "input_required"`, and the client retries the original
  request with the answers attached. Every result carries `resultType`.
- **`subscriptions/listen`** replaces the HTTP GET stream and
  `resources/subscribe`: one long-lived stream for opted-in change
  notifications. Request-scoped notifications (progress) still ride the
  response stream of the request they belong to.
- **Tasks** (long-running work) moved out of the core into the
  `io.modelcontextprotocol/tasks` extension, polled via `tasks/get`.
- **Transports:** stdio for co-located servers, **Streamable HTTP** for
  networked ones. Streamable HTTP POSTs carry `MCP-Protocol-Version`,
  `Mcp-Method`, and `Mcp-Name` headers so gateways route and meter without
  parsing bodies; stream resumability (`Last-Event-ID`) is removed — a broken
  stream means re-issuing the request.

**Deprecated — do not adopt in new code** (they keep working for a minimum
twelve-month window under the feature lifecycle policy, then become eligible
for removal):

| Deprecated | Migrate to |
|---|---|
| Roots | pass directories or files as tool parameters, resource URIs, or server configuration |
| Sampling | call the model provider's API directly |
| Logging (`logging/setLevel`, server log messages) | stderr on stdio; OpenTelemetry for networked servers |
| HTTP+SSE transport (deprecated since 2025-03-26) | Streamable HTTP |
| OAuth Dynamic Client Registration | Client ID Metadata Documents |

Peers on handshake-based revisions (2025-11-25 and earlier) are handled by the
revision's backward-compatibility rules; the official SDKs negotiate down
automatically. Authorization for HTTP transports is its own reference — see
<ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::orchestration@agentic_development/reference/mcp_authorization.md" />.

## A2A: the current revision

A2A reached its first stable release, **1.0.0**, on 2026-03-12; the current
release is **1.0.1** (2026-05-28, specification fixes). Compatibility is
tracked by Major.Minor, carried in the `A2A-Version` service parameter.

- **AgentCard / AgentSkill** — the agent publishes a capability card at
  `/.well-known/agent-card.json` describing identity, version, supported
  interfaces, capabilities (`streaming`, `pushNotifications`,
  `extendedAgentCard`), security schemes, and skills (id, description, tags,
  input/output modes). Cards may be **signed** (JWS over the JCS-canonicalized
  card); verify a signature before trusting a card fetched across a trust
  boundary. Authenticated callers may fetch an **extended card** with
  additional skills.
- **Protocol bindings** — JSON-RPC, gRPC, and HTTP+JSON/REST are all
  normative; a card declares which interfaces it serves.
- **Task state machine** — a closed status set: submitted → working →
  completed | failed | canceled | rejected (terminal), with input-required and
  auth-required as the interrupted states. `input-required` is the standard
  pause for user confirmation or clarification and `auth-required` for a
  credential the agent needs — model them, do not invent a bespoke
  confirmation channel.
- **Artifact / Part streaming** — domain payloads stream as typed Artifacts
  composed of Parts, delivered over Server-Sent Events
  (`SendStreamingMessage`, `SubscribeToTask`) or the gRPC stream, with push
  notifications for disconnected callers.

## Schema governance: vendor, never hand-write

Both protocols publish their canonical schemas — MCP a TypeScript schema with
generated JSON Schema, A2A Protocol Buffers with generated JSON. Vendor the
official specification files at a **tagged revision** (`2026-07-28`,
`v1.0.1`) into the contract tree and track upstream revisions, rather than
re-declaring approximate schemas by hand — approximation is where wire drift
and silent incompatibility begin. A revision bump is a deliberate change that
names the old and new revisions and walks the upstream changelog's
deprecations. Compatibility gating for the vendored schemas (lint,
breaking-change detection against upstream) is the repository's existing
contract-toolchain concern, owned by the language profiles; this profile owns
only the vendoring posture.

## Related

- <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::orchestration@agentic_development/reference/mcp_authorization.md" />
- <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::orchestration@agentic_development/reference/skills.md" />
- <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::orchestration@agentic_development/reference/mcp_servers_go.md" />
- <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::orchestration@agentic_development/reference/interface_axis_split.md" />

## Sources

- https://modelcontextprotocol.io/specification/2026-07-28/changelog — MCP key changes since 2025-11-25 (stateless core, `server/discover`, MRTR, `subscriptions/listen`, tasks extension, deprecations), revision 2026-07-28, accessed 2026-11-22
- https://modelcontextprotocol.io/specification/versioning — current revision 2026-07-28, per-request negotiation, Deprecated feature state, accessed 2026-11-22
- https://blog.modelcontextprotocol.io/posts/2026-07-28/ — release announcement, Tier 1 SDK support and migration themes, 2026-07-28, accessed 2026-11-22
- https://a2a-protocol.org/latest/specification/ — A2A specification 1.0 (task states, bindings, agent card discovery and signing, extended card, `A2A-Version`), accessed 2026-11-22
- https://github.com/a2aproject/A2A/releases — A2A v1.0.0 (2026-03-12) and v1.0.1 (2026-05-28), accessed 2026-11-22
- https://a2a-protocol.org/latest/topics/a2a-and-mcp/ — A2A and MCP as complements (tools versus opaque agents), accessed 2026-11-22
- https://aaif.io/about — AAIF hosted projects (MCP, goose, AGENTS.md, A2A, agentgateway), accessed 2026-11-22
