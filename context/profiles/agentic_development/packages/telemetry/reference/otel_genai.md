# OpenTelemetry Core and the GenAI Conventions

OpenTelemetry core is the stable base; GenAI semantic conventions are experimental behind OTEL_SEMCONV_STABILITY_OPT_IN — volatility awareness and the thin adapter rule.

AI telemetry has a stability problem: the carrier is mature, the vocabulary
is not. The convention that keeps you safe is to layer — stable base,
neutral semantics, experimental attributes isolated behind flags.

## The stable base

OpenTelemetry core — spans, resources, context propagation, OTLP export —
is settled and universally supported. Instrument on it unconditionally: one
span per agent step, one per tool invocation, one per model call, exported
over OTLP to whatever backend the platform reference selects. Nothing in
this section is experimental.

## GenAI semantic conventions: bleeding edge, and volatile

The OpenTelemetry GenAI semantic conventions (`gen_ai.*` attributes: model
identity, token consumption, prompt/completion shapes, latency, tool
selection) are correct in spirit and ahead of spec stability. As of 2026:

- every `gen_ai.*` attribute is still marked **Development** stability;
- the conventions recently moved into their own repository without a tagged
  release;
- attribute names and shapes may change between versions;
- experimental versions are consumed via `OTEL_SEMCONV_STABILITY_OPT_IN`.

None of this makes the conventions a wrong choice — it makes them a bet that
must be contained.

## The thin adapter rule

Contain the bet structurally:

1. Emit every `gen_ai.*` attribute from **one thin adapter package**,
   clearly marked experimental. No experimental attribute name appears in
   business or agent code.
2. Gate the adapter behind a **feature flag**, so switching vocabulary or
   disabling the convention entirely is a configuration change.
3. **Pin the convention version** the adapter emits against, and record the
   pin where dependencies are pinned — an unpinned experimental convention
   drifts under you silently.
4. Keep the adapter's internal interface semantic ("model call", "tool
   result"), so a future convention rename is an adapter edit, not a
   codebase sweep.

When the conventions reach stable, the migration cost is one package. When
they change again, it still is.

## Related

- <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::telemetry@agentic_development/reference/openinference.md" />
- <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::telemetry@agentic_development/reference/platforms.md" />
- <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::orchestration@agentic_development/reference/mcp_servers_go.md" />
