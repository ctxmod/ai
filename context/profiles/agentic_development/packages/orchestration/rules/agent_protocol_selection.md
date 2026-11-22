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
discovery, invocation, and dynamic resources. Horizontally — client ↔ agent,
and agent ↔ agent — use A2A: sessions, capability cards, the task state
machine, and streamed artifacts.

Vendor the official protocol specifications rather than re-declaring
approximate schemas; both protocols publish canonical Protocol Buffers and
JSON Schema. Compatibility gating for vendored schemas follows the
repository's existing contract-toolchain conventions, owned by the language
profiles.

- the protocol division, message surface, and schema governance — <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::orchestration@agentic_development/reference/protocols_mcp_a2a.md" />
- runtime capabilities that shape protocol choices, and hosted model access — <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::orchestration@agentic_development/reference/runtimes.md" />
- the same axis split applied to AI UI kits — <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::orchestration@agentic_development/reference/interface_axis_split.md" />
