# Agent Runtimes

What an agent runtime owns, a Python agent-SDK-style runtime as one worked example, hosted model access, and runtime-agnostic selection criteria.

An agent runtime owns the machinery around the model: the reasoning loop,
tool mounting and dispatch, session state, protocol endpoints, and the
telemetry export hooks. Everything in this profile — skills, protocols,
telemetry, evaluation — is defined runtime-agnostically; a runtime is an
interchangeable implementation of those patterns.

## Runtime anatomy

Whatever the language, a runtime answers the same five questions:

1. **Loop** — how turns are driven: model call, tool dispatch, result
   feeding, termination.
2. **Mounting** — how tools and skills attach and detach per session, and
   how tool schemas reach the model.
3. **State** — how session history, artifacts, and task state persist and
   resume.
4. **Endpoints** — which protocol surfaces it serves (client ↔ agent
   horizontally, agent ↔ tools vertically) and over which transports.
5. **Observability** — where spans and events are emitted for export.

## A Python agent-SDK-style runtime as one example

Python agent-SDK-style runtimes (Google ADK is the worked example; it also
ships Java, Go, TypeScript, and Kotlin editions) ship the patterns of this
profile as first-class primitives: progressive disclosure via skill listing
and loading, the A2A task lifecycle with `AgentCard`/`AgentSkill` capability
discovery, and native tool-protocol client support for mounting external
tool servers. They are illustrative of the shape — an ecosystem example, not
an endorsement over other runtimes with equivalent surfaces.

## Hosted model access

Runtimes commonly resolve model calls through a hosted provider endpoint
(Vertex AI is one example deployment shape) rather than self-hosted
inference. Two consequences: model identity and version become configuration
that evaluation gates must pin (a silent model bump invalidates gate
baselines), and credentials follow the repository's IAM conventions —
workload-identity federation and short-lived tokens are the delivery/IAM
profiles' concern and are not restated here.

## Selection criteria

When choosing a runtime, score it against the patterns rather than the
marketing:

- protocol coverage: serves the client-boundary protocol and speaks the
  tool-boundary protocol as a client, at the current revisions (A2A 1.x, MCP
  2026-07-28) while negotiating down for older peers;
- dynamic mounting: per-session tool/skill attach and detach, done in the
  runtime — the current MCP revision keeps server tool lists
  connection-independent, so a runtime that relies on per-session server
  lists is building on a removed feature;
- observability hooks: span export through standard OTLP with the semantic
  layer this profile standardizes on;
- evaluation fit: session replay or deterministic sampling enough to feed
  the eval pipeline;
- deployment shape: how it runs as a service in the repository's process
  model.

A runtime missing a pattern is extendable; a runtime hostile to a pattern
(the loop cannot be instrumented, mounting is static) is a dead end.

## Related

- <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::orchestration@agentic_development/reference/protocols_mcp_a2a.md" />
- <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::orchestration@agentic_development/reference/skills.md" />
- <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::telemetry@agentic_development/reference/otel_genai.md" />

## Sources

- https://pypi.org/project/google-adk/ — ADK Python 2.15.0 (2026-11-20), A2A and MCP extras, links to the Java, Kotlin, Go, and TypeScript editions, accessed 2026-11-22
- https://modelcontextprotocol.io/specification/2026-07-28/changelog — connection-independent list endpoints, accessed 2026-11-22
