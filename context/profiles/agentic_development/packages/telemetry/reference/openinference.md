# OpenInference: the Backend-Neutral Semantic Layer

OpenInference as the backend-neutral semantic layer for model, retrieval, and tool spans — the portability layer under the platforms.

Where `gen_ai.*` is an upstream standards bet, OpenInference is the working
vocabulary: a set of well-scoped semantic conventions for AI spans, carried
on standard OTLP and designed to be read by any compliant tool.

## What it defines

OpenInference names the span kinds an AI system actually produces through a
required `openinference.span.kind` attribute (distinct from OpenTelemetry's
own span kind): `LLM`, `EMBEDDING`, `RETRIEVER`, `RERANKER`, `TOOL`, `AGENT`,
`CHAIN`, `GUARDRAIL`, `EVALUATOR`, `PROMPT`, and others. Attributes cover
prompts and completions (`llm.input_messages`, `llm.output_messages`),
retrieved documents with ids and scores (`retrieval.documents`), reranker
input and output, token counts including cache and reasoning detail
(`llm.token_count.*`), and error states. The scope is deliberate: enough structure that tracing
tooling can render a session meaningfully, without trying to standardize
the whole AI stack.

## Why it is the default layer

- **Backend-neutral** — spans emit over standard OTLP; any OpenInference-
  aware reader (an open platform, a hosted vendor, a local debug UI) renders
  them. Instrument once, read anywhere.
- **Lingua franca** — the tracing tooling ecosystem already speaks it, so
  spans are immediately useful to the platforms in the landscape reference
  without per-vendor exporters.
- **Scoped, not frozen** — its vocabulary is maintained by a project whose
  purpose is exactly this layer, generated from a machine-readable registry.
  It is not a stability guarantee: the specification itself marks its
  attributes Development and deprecates names (`llm.function_call` gave way
  to `message.tool_calls`), so the thin adapter rule applies to it as much
  as to `gen_ai.*`.

## Relationship to the GenAI conventions

The two layers answer different questions:

| | OpenInference | `gen_ai.*` conventions |
|---|---|---|
| Role | working semantic layer | upstream standards bet |
| Stability | Development, scoped and tool-adopted | Development, volatile, first standalone release v0.1.0 |
| Posture | default; emit now | behind a stability opt-in flag |

The two are converging at the backend rather than at the emitter: Phoenix
(since 15.10, May 2026) and Arize AX translate incoming `gen_ai.*` spans into
OpenInference on ingest, MLflow ingests `gen_ai.*` over OTLP and can export
in it, and other platforms map both. Ingest-side translation lowers the cost
of choosing wrong, but it is lossy at the edges — check that the attributes
your evaluations read (retrieved documents, tool calls, token detail)
survive the translation on the backend you run.

Do not emit both vocabularies for the same span from the same code path —
pick per adapter, and keep the choice inside the telemetry adapter package
(the thin adapter rule in the GenAI reference applies equally here). The
retrieval span vocabulary is what makes retrieval-quality evaluation
possible downstream: the RAG profile's contextual metrics read spans shaped
by this layer, in prose terms, from production traces.

## Related

- <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::telemetry@agentic_development/reference/otel_genai.md" />
- <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::telemetry@agentic_development/reference/platforms.md" />

## Sources

- https://openinf.dev/spec/attributes/semantic-conventions.html — OpenInference span kinds and attributes, Development marking, deprecated function-call attributes, accessed 2026-11-22
- https://www.arize.com/docs/phoenix/release-notes/05-2026/05-15-2026-otel-semconv-conversion.md — Phoenix 15.10 converts `gen_ai.*` to OpenInference on ingest, 2026-05-15, accessed 2026-11-22
- https://arize.com/blog/arize-ax-opentelemetry-genai-semantic-conventions/ — Arize AX normalizes `gen_ai.*` spans to OpenInference, August 2026, accessed 2026-11-22
- https://mlflow.org/docs/latest/genai/tracing/opentelemetry/genai-semconv/ — MLflow `gen_ai.*` OTLP ingest and export (3.11+), accessed 2026-11-22
