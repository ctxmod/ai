---
paths:
  - "services/*"
  - "services/*/*"
  - "services/*/*/*"
  - "services/*/*/*/*"
  - "*/services/*"
  - "*/services/*/*"
  - "*/services/*/*/*"
  - "*/*/services/*"
  - "*/*/services/*/*"
  - "*/*/*/services/*"
---

# Agent Protocol Selection

Two open protocols divide the agent stack, and inventing either boundary ad
hoc is the anti-pattern. Vertically — agent ↔ tools — use MCP: tool
discovery, invocation, resources, and prompts. Horizontally — client ↔ agent,
and agent ↔ agent, where the callee is an opaque agent rather than a schema'd
capability — use A2A: capability cards, the task state machine, and streamed
artifacts.

Build against the current revisions — MCP 2026-07-28 (stateless: no
handshake, no protocol sessions, `server/discover`) and A2A 1.x — and never
adopt features the revision deprecates (MCP Roots, Sampling, Logging, the
HTTP+SSE transport, Dynamic Client Registration). A networked MCP server is
an OAuth 2.1 resource server that accepts only tokens issued for it.

Vendor the official protocol specifications at a tagged revision rather than
re-declaring approximate schemas. Compatibility gating for vendored schemas
follows the repository's existing contract-toolchain conventions, owned by
the language profiles.

- the protocol division, message surface, and schema governance — <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::orchestration@agentic_development/reference/protocols_mcp_a2a.md" />
- server and client obligations for MCP authorization — <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::orchestration@agentic_development/reference/mcp_authorization.md" />
- runtime capabilities that shape protocol choices, and hosted model access — <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::orchestration@agentic_development/reference/runtimes.md" />
- the same axis split applied to AI UI kits — <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::orchestration@agentic_development/reference/interface_axis_split.md" />
