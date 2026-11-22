# OpenTelemetry Core and the GenAI Conventions

OpenTelemetry core is the stable base; the GenAI semantic conventions are Development-stability, now versioned in their own repository — volatility awareness, the span shapes, content capture, and the thin adapter rule.

AI telemetry has a stability problem: the carrier is mature, the vocabulary
is not. The convention that keeps you safe is to layer — stable base,
neutral semantics, experimental attributes isolated behind flags.

## The stable base

OpenTelemetry core — spans, resources, context propagation, OTLP export —
is settled and universally supported. Instrument on it unconditionally: one
span per agent step, one per tool invocation, one per model call, exported
over OTLP to whatever backend the platform reference selects. Nothing in
this section is experimental. Propagate W3C trace context across every hop,
including tool-protocol calls: MCP 2026-07-28 documents `traceparent`,
`tracestate`, and `baggage` keys in a request's `_meta`, so a tool server's
spans join the calling agent's trace.

## GenAI semantic conventions: bleeding edge, and volatile

The OpenTelemetry GenAI semantic conventions (`gen_ai.*`) are correct in
spirit and ahead of spec stability. As of 2026-11-22:

- they moved out of the core semantic-conventions repository into the
  dedicated `open-telemetry/semantic-conventions-genai` repository, whose
  **first standalone release is v0.1.0** (November 2026, built on core
  semantic conventions v1.45.0); the opentelemetry.io GenAI pages are frozen
  "moved" stubs;
- every `gen_ai.*` span, attribute, metric, and event is still
  **Development** stability — only borrowed general attributes
  (`error.type`, `server.address`, `server.port`) are Stable;
- names still change between versions — v0.1.0 alone renamed
  `gen_ai.workflow.name` to `gen_ai.agent.name` and
  `gen_ai.usage.cache_creation.input_tokens` to
  `gen_ai.usage.cache_write.input_tokens`, folded `invoke_workflow` into the
  internal agent span, and replaced `gen_ai.client.token.usage` with
  separate input and output token histograms;
- instrumentation libraries switch from the older emitted shape to the latest
  experimental one through `OTEL_SEMCONV_STABILITY_OPT_IN=gen_ai_latest_experimental`;
  instrumentations that never opt in keep emitting an older shape.

None of this makes the conventions a wrong choice — it makes them a bet that
must be contained.

## The span shapes worth knowing

The vocabulary is volatile, but its structure has settled:

- **Model calls** — `chat`, `text_completion`, `generate_content` (CLIENT
  spans named `{gen_ai.operation.name} {gen_ai.request.model}`), requiring
  `gen_ai.operation.name` and `gen_ai.provider.name` and carrying request and
  response model, token usage, and finish reasons.
- **Agent steps** — `invoke_agent`, CLIENT for a remote agent service and
  INTERNAL for an agent in-process, parenting the model-call and tool spans of
  that step; `gen_ai.agent.name` identifies the agent.
- **Tool calls** — `execute_tool {gen_ai.tool.name}` (INTERNAL), with
  `gen_ai.tool.call.id` required when the provider supplies one. Loading a
  skill and reading a skill resource are `execute_tool` variants carrying
  `gen_ai.skill.name` and `gen_ai.skill.resource.name` — the hook for
  observing skill activation in production.
- **Retrieval and embeddings** — `retrieval` (with `gen_ai.retrieval.top_k`)
  and `embeddings` spans, plus memory-store operations.
- **MCP** — dedicated conventions for MCP client and server spans and
  metrics, with tool arguments and results as opt-in server-span attributes.

## Content capture is opt-in

Prompts, completions, system instructions, and tool arguments are sensitive
and large, so instrumentations record none of them by default.
The conventions define `OTEL_INSTRUMENTATION_GENAI_CAPTURE_CONTENT` with
`NO_CONTENT` (default), `SPAN_ONLY`, `EVENT_ONLY`, or `SPAN_AND_EVENT`;
shipped instrumentations may lag the name (the Python contrib packages read
`OTEL_INSTRUMENTATION_GENAI_CAPTURE_MESSAGE_CONTENT`, and honor the modes only
with the latest-experimental opt-in), so check each instrumentation's
documentation and set the variable it actually reads. Record content on spans only
in pre-production; in production, **store content externally and record
references**, which the conventions name as the recommended production
pattern. Content capture is a data-governance decision — retention, access,
and redaction follow the repository's privacy conventions, not the tracer's
defaults.

## The thin adapter rule

Contain the bet structurally:

1. Emit every `gen_ai.*` attribute from **one thin adapter package**,
   clearly marked experimental. No experimental attribute name appears in
   business or agent code.
2. Gate the adapter behind a **feature flag**, so switching vocabulary or
   disabling the convention entirely is a configuration change; set the
   instrumentation opt-in (`OTEL_SEMCONV_STABILITY_OPT_IN`) and the
   content-capture mode in the same configuration, never in code.
3. **Pin the convention version** the adapter emits against — a
   `semantic-conventions-genai` release, now that one exists — and record the
   pin where dependencies are pinned; an unpinned experimental convention
   drifts under you silently. A pin bump walks the upstream changelog's
   renames.
4. Keep the adapter's internal interface semantic ("model call", "tool
   result"), so a future convention rename is an adapter edit, not a
   codebase sweep.

When the conventions reach stable, the migration cost is one package. When
they change again, it still is.

## Related

- <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::telemetry@agentic_development/reference/openinference.md" />
- <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::telemetry@agentic_development/reference/platforms.md" />
- <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::orchestration@agentic_development/reference/mcp_servers_go.md" />

## Sources

- https://opentelemetry.io/docs/specs/semconv/gen-ai/ — GenAI pages marked moved to the dedicated repository, accessed 2026-11-22
- https://github.com/open-telemetry/semantic-conventions-genai/releases — v0.1.0, first standalone release (2026-11-21), accessed 2026-11-22
- https://raw.githubusercontent.com/open-telemetry/semantic-conventions-genai/main/CHANGELOG.md — v0.1.0 changes since core v1.41.0 (renames, agent and tool span changes, token metrics, MCP span changes), accessed 2026-11-22
- https://raw.githubusercontent.com/open-telemetry/semantic-conventions-genai/main/docs/gen-ai/gen-ai-spans.md — Development status, span definitions, content-capture configuration and recommended external-storage pattern, accessed 2026-11-22
- https://modelcontextprotocol.io/specification/2026-07-28/changelog — trace context propagation in `_meta` (SEP-414), accessed 2026-11-22
- https://raw.githubusercontent.com/open-telemetry/semantic-conventions-genai/main/docs/gen-ai/client-inference.md — inference span name, kind, operation names, required attributes, accessed 2026-11-22
- https://raw.githubusercontent.com/open-telemetry/semantic-conventions-genai/main/docs/gen-ai/gen-ai-agent-spans.md — `invoke_agent` internal span, `create_agent`, `invoke_runner`, accessed 2026-11-22
- https://raw.githubusercontent.com/open-telemetry/semantic-conventions-genai/main/docs/gen-ai/execute-tool.md — `execute_tool` span with skill-load and skill-resource variants, accessed 2026-11-22
- https://opentelemetry-python-contrib.readthedocs.io/en/latest/instrumentation-genai/openai.html — `OTEL_SEMCONV_STABILITY_OPT_IN=gen_ai_latest_experimental` opt-in (default stays on older conventions) and `OTEL_INSTRUMENTATION_GENAI_CAPTURE_MESSAGE_CONTENT` modes in a shipped instrumentation, accessed 2026-11-22
