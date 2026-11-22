# The Interface Axis Split for AI UI Kits

AI UI kits split into a framework-free protocol axis and a presentation axis — bring-your-own runtime, open registries with namespaced extension keys, named fallback branches.

An AI-facing UI kit — the component layer that renders conversations, tool
activity, and agent state — spans two orthogonal axes. Kits that fuse them
become framework lock-in; kits that split them stay composable.

## The two axes

- **Protocol axis** — the semantic types that flow between the application
  and the model: conversation turns, tool calls, approvals, plans, activity
  events, artifacts. This axis is **framework-free**: it is a typed protocol,
  not a rendering decision, and it must not know which UI framework draws it.
- **Presentation axis** — components, styling, layout, interaction patterns.
  This axis consumes protocol events and renders them.

The dependency is **one-directional**: the presentation layer depends on the
protocol layer, never the reverse. A protocol event cannot require a
component to exist.

## Bring-your-own runtime

The kit never reaches out to a specific agent runtime or model backend.
**Registration flows into the kit** — the application connects its runtime,
transport, and credential handling, and the kit consumes the resulting event
streams. A kit that ships with a hard-wired backend has collapsed the axis
split at the worst possible seam.

## Open registries with namespaced extension keys

Protocol events and capabilities live in open registries: unknown event
types and unknown fields are preserved and rendered generically, never
rejected. Extensions use **namespaced keys** — `x.<org>.<name>` — so
organizations add semantics without colliding with the core vocabulary or
each other. A registry that drops what it does not recognize makes every
extension a fork.

## Named fallback branches

Every optional capability has an explicit **named fallback with a documented
activation condition**: streaming unavailable → incremental polling; skill
mounting unavailable → instruction-only degradation; attachments unsupported
→ reference links. The fallback is a first-class branch in the protocol, not
a silent failure in a component. Silent degradation is how AI interfaces
fail invisibly.

## Relation to the agent protocol split

This is the same division the agent protocols make — a vertical
tool-boundary protocol and a horizontal client-boundary protocol, each with
its own schema governance — applied at the UI layer. See
`reference/orchestration/protocols-mcp-a2a.md` in this profile for the
protocol-level version.

## Related

- <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::orchestration@agentic_development/reference/protocols_mcp_a2a.md" />
- <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::orchestration@agentic_development/reference/skills.md" />
- <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::evaluation@agentic_development/reference/observation_not_verdict.md" />
