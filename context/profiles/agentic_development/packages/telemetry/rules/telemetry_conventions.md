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
and volatile: emit them only from a thin, clearly-marked adapter package,
behind a stability opt-in flag (`OTEL_SEMCONV_STABILITY_OPT_IN`), pinned to a
convention version. Never scatter experimental attributes through business
code.

- the stability posture and the thin adapter rule — <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::telemetry@agentic_development/reference/otel_genai.md" />
- OpenInference span semantics for model, retrieval, and tool spans — <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::telemetry@agentic_development/reference/openinference.md" />
- platform landscape, license and governance awareness, the adapter pattern — <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::telemetry@agentic_development/reference/platforms.md" />
