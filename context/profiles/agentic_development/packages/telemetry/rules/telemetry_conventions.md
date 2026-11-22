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
  - "pkg/*"
  - "pkg/*/*"
  - "pkg/*/*/*"
  - "pkg/*/*/*/*"
  - "*/pkg/*"
  - "*/pkg/*/*"
  - "*/pkg/*/*/*"
  - "*/*/pkg/*"
  - "*/*/pkg/*/*"
  - "*/*/*/pkg/*"
---

# AI Telemetry Conventions

Treat OpenTelemetry core (spans, resources, OTLP export) as the stable base
and instrument on it unconditionally. Layer OpenInference semantics on top
for model calls, retrieval, and tool invocations — it is the backend-neutral
lingua franca of AI tracing tooling.

The OpenTelemetry GenAI conventions (`gen_ai.*`) remain Development-stability
and volatile — now versioned in their own repository, first released as
v0.1.0: emit them only from a thin, clearly-marked adapter package, behind a
feature flag and the instrumentation opt-in
(`OTEL_SEMCONV_STABILITY_OPT_IN=gen_ai_latest_experimental`), pinned to a
convention release. Never scatter experimental attributes through business
code. Prompt and completion content is off by default; capture it by explicit
configuration, and in production store it externally and record references.
Propagate trace context into tool-protocol calls so tool-server spans join
the agent's trace.

- the stability posture and the thin adapter rule — <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::telemetry@agentic_development/reference/otel_genai.md" />
- OpenInference span semantics for model, retrieval, and tool spans — <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::telemetry@agentic_development/reference/openinference.md" />
- platform landscape, license and governance awareness, the adapter pattern — <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::telemetry@agentic_development/reference/platforms.md" />
