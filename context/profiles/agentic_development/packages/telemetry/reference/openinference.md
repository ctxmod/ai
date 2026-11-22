# OpenInference: the Backend-Neutral Semantic Layer

OpenInference as the backend-neutral semantic layer for model, retrieval, and tool spans — the portability layer under the platforms.

Where `gen_ai.*` is an upstream standards bet, OpenInference is the working
vocabulary: a set of well-scoped semantic conventions for AI spans, carried
on standard OTLP and designed to be read by any compliant tool.

## What it defines

OpenInference names the span kinds an AI system actually produces — model
/ LLM calls, retrieval, embedding, tool invocation, agent steps — with
attributes for prompts, completions, retrieved documents, token counts, and
error states. The scope is deliberate: enough structure that tracing
tooling can render a session meaningfully, without trying to standardize
the whole AI stack.

## Why it is the default layer

- **Backend-neutral** — spans emit over standard OTLP; any OpenInference-
  aware reader (an open platform, a hosted vendor, a local debug UI) renders
  them. Instrument once, read anywhere.
- **Lingua franca** — the tracing tooling ecosystem already speaks it, so
  spans are immediately useful to the platforms in the landscape reference
  without per-vendor exporters.
- **Stable in the way that matters** — its vocabulary is scoped and
  versioned by a project whose purpose is exactly this layer, not a
  moving standards track.

## Relationship to the GenAI conventions

The two layers answer different questions:

| | OpenInference | `gen_ai.*` conventions |
|---|---|---|
| Role | working semantic layer | upstream standards bet |
| Stability | scoped, tool-adopted | Development, volatile |
| Posture | default; emit now | behind a stability opt-in flag |

Do not emit both vocabularies for the same span from the same code path —
pick per adapter, and keep the choice inside the telemetry adapter package
(the thin adapter rule in the GenAI reference applies equally here). The
retrieval span vocabulary is what makes retrieval-quality evaluation
possible downstream: the RAG profile's contextual metrics read spans shaped
by this layer, in prose terms, from production traces.

## Related

- <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::telemetry@agentic_development/reference/otel_genai.md" />
- <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::telemetry@agentic_development/reference/platforms.md" />
