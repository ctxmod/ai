# Skills and the Dynamic Tool Surface

Always-on base tools vs dynamically mounted skill bundles — the descriptor pattern (injected instructions plus mounted tool schemas), progressive disclosure, and the skill lifecycle.

An agent that loads every capability it might ever need pays for all of it in
every session: context bloat, tool-selection confusion, and prompt dilution.
The corrective architecture keeps a deliberately small always-on core and
mounts domain capability on demand.

## Base tools vs dynamic skills

**Base tools** mount at bootstrap and stay for every session — the small set
almost any conversation needs: state query/update for the domain's primary
entity, general research or retrieval, and user-preference lookup.

**Skills** are self-contained capability bundles mounted when a session
actually calls for them, either intent-detected (the platform recognizes the
request shape) or user-toggled (the caller selects capability explicitly).
Both paths surface an activation event so the caller's UI can show what is
mounted and why.

## The skill descriptor pattern

Each skill is a descriptor package with three parts:

1. **Identity metadata** — id, name, description, and the tool names it
   provides, exposed through the standard capability-discovery surface of
   the agent protocol.
2. **Injected instructions** — domain rules spliced into the system prompt on
   activation: the heuristics, constraints, and procedures the model should
   follow for this capability.
3. **Mounted tool schemas** — the tool-protocol servers (and their tool
   schemas) registered into the session for the skill's duration.

The descriptor pattern is what makes skills auditable and testable units:
a skill is data plus servers, inspectable before it ever runs.

## Progressive disclosure

Instructions and tools enter the context **only when relevant** — that is
the whole mechanism. Progressive disclosure keeps the base session cheap and
focused, and it is why skill boundaries must be genuine: a skill that needs
to be mounted "just in case" is a base tool that has not admitted it yet.

## Evaluating dynamic routing

Because activation can be automatic, routing is itself a behavior under test:
skill-selection precision — when a prompt calls for a capability, does the
agent activate the right skill — is gated statistically over a scenario
suite, not trusted by construction. See
`reference/evaluation/gates.md` for the gate mechanics.

## Lifecycle

Skills move through declare → register → activate → evaluate → deprecate.
Declaration and registration are versioned with the codebase; activation is
per-session; evaluation rides the standard eval pipeline; deprecation
removes the descriptor and its servers together, so no orphaned tool schemas
linger discoverable.

## Related

- <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::orchestration@agentic_development/reference/mcp_servers_go.md" />
- <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::orchestration@agentic_development/reference/protocols_mcp_a2a.md" />
- <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::evaluation@agentic_development/reference/gates.md" />
