# Agent Protocols: MCP and A2A

The protocol division of labor — MCP vertically for agent-to-tools, A2A horizontally for client-to-agent — and vendored-schema governance for both.

The industry has consolidated on two canonical open protocols — both under
the Linux Foundation — covering the two boundaries in an agent stack. The
architectural mistake to avoid is inventing either boundary ad hoc, or using
one protocol for both jobs.

## The division of labor

- **MCP (Model Context Protocol) is vertical: agent ↔ tools.** It defines
  how an agent discovers tools (`tools/list`), invokes them (`tools/call`),
  and reads dynamic contextual resources. Transport is JSON-RPC over stdio
  (co-located servers) or streamable HTTP/SSE (networked servers).
- **A2A (Agent2Agent) is horizontal: client ↔ agent, and agent ↔ agent.** It
  defines stateful multi-turn sessions, progressive capability discovery,
  and a standardized task lifecycle between the caller and the agent.

## The three-tier mapping

A typical stack routes each boundary through its own protocol:

```
client (UI / service)
   │  A2A — sessions, capability discovery, task lifecycle
   ▼
agent runtime
   │  MCP — tool discovery, invocation, resources
   ▼
tool servers
```

## The A2A surface in brief

- **AgentCard / AgentSkill** — the agent publishes a capability card
  describing identity, version, supported interfaces, and a list of skills
  (id, description, tags, input/output modes). Discovery reads the card;
  nothing is hard-coded client-side.
- **Task state machine** — a task moves through a closed status set:
  submitted → working → (input-required) → completed | failed | canceled.
  `input-required` is the standard pause for user confirmation or
  clarification — model it, do not invent a bespoke confirmation channel.
- **Artifact / Part streaming** — domain payloads stream as typed Artifacts
  (named, MIME-typed) composed of Parts, delivered over SSE or an
  equivalent streaming transport as task updates.

## Schema governance: vendor, never hand-write

Both protocols publish their canonical schemas (Protocol Buffers and JSON
Schema). Vendor the official specification files into the contract tree and
track upstream versions, rather than re-declaring approximate schemas by
hand — approximation is where wire drift and silent incompatibility begin.
Compatibility gating for the vendored schemas (lint, breaking-change
detection against upstream) is the repository's existing contract-toolchain
concern, owned by the language profiles; this profile owns only the
vendoring posture.

## Related

- <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::orchestration@agentic_development/reference/skills.md" />
- <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::orchestration@agentic_development/reference/mcp_servers_go.md" />
- <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::orchestration@agentic_development/reference/interface_axis_split.md" />
