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

# Skill Lifecycle

Keep the base context small: a handful of always-on base tools (state
query/update, research, user preferences) mounted at bootstrap. Domain
capability ships as skill bundles mounted on demand — each a self-contained
descriptor carrying identity metadata, injected domain instructions, and the
tool schemas it registers into the session.

Activation is intent-detected or user-toggled, and activation events are
surfaced to the caller. Skill-routing precision is itself evaluated as a
statistical gate over a scenario suite — routing is not trusted by
construction.

- base tools vs dynamic skills, the descriptor pattern, progressive disclosure — <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::orchestration@agentic_development/reference/skills.md" />
- building and dynamically mounting the tool servers skills depend on — <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::orchestration@agentic_development/reference/mcp_servers_go.md" />
