# Observability Platforms for AI Systems

The observability platform landscape with license and governance awareness — traces, datasets, experiments, and judge evals in one loop, behind a thin feature-flagged adapter.

A tracing backend for AI systems is more than span storage. The mature
platforms join four things in one loop: **hierarchical traces** for every
agent/tool call, **datasets** curated from what the traces saw,
**experiments** (runs against datasets), and **judge evaluations** scored on
those runs. That closed loop is what feeds the production-trace → dataset →
eval cycle, which is why platform choice is an evaluation decision, not just
an ops one.

## The landscape, with governance awareness

- **Langfuse** — the current default open platform: MIT-licensed core,
  self-hostable, with native dataset/experiment/eval surface and
  OpenTelemetry-compatible ingest. Its agent-eval loop (traces + datasets +
  LLM-as-judge in one place) is the pattern the loop above describes.
  Governance changed in January 2026: ClickHouse acquired Langfuse, stating
  no licensing change for the core and continued self-hosting (Langfuse
  already ran on ClickHouse). Track that commitment like any vendor-owned
  open project.
- **Phoenix (Arize)** — capable and widely adopted, but **source-available
  under the Elastic License 2.0, not OSI open source**. Dynatrace announced
  its acquisition of Arize on 2026-08-13 and reported it completed in early
  October 2026, with Phoenix and Arize AX to continue; the open project's
  long-term governance under an APM vendor is the open question to watch.
  OpenInference itself remains backend-neutral; the governance question is
  about the platform, not the convention.
- **MLflow** — Apache-2.0 and Linux Foundation-hosted, with tracing,
  datasets, and judge-based evaluation; its server ingests `gen_ai.*` spans
  over OTLP and its SDK can export them (3.11+). The vendor-neutral-governance
  option when that grid row outweighs feature depth.
- **Hosted commercial alternatives** — LangSmith, Braintrust, and lighter
  entries like Laminar trade self-hosting for operational convenience.

Evaluate every platform on the same grid: **license** (OSS vs
source-available), **governance trajectory** (who owns it, who is buying
whom), **self-host capability** (can traces stay in-cluster),
**ingest compatibility** (standard OTLP with the OpenInference layer and
`gen_ai.*` both mapped, not a proprietary SDK requirement), and
**dataset/experiment API surface**
(can the eval pipeline read and write datasets programmatically).

## The thin experimental adapter pattern

Whatever the choice, no platform SDK call appears outside one narrow
internal interface: agent and eval code speak semantic operations ("start a
trace", "fetch flagged traces", "log an experiment row"), and the adapter
implements them against the platform, behind **feature flags**. Backends
then swap — or run side-by-side during migration — without touching agent
code. This is the same containment discipline as the telemetry adapter
rule, applied one level up.

## Self-hosting and the loop

Self-hosting the platform inside the cluster keeps real-time trace capture
close to the services and makes the feedback loop cheap: flagged production
edge cases flow from the platform's dataset surface back into the eval
tree's golden datasets without crossing a trust boundary. The loop mechanics
are the feedback-loop reference's; the platform is its plumbing.

## Related

- <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::telemetry@agentic_development/reference/otel_genai.md" />
- <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::telemetry@agentic_development/reference/openinference.md" />
- <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::evaluation@agentic_development/reference/feedback_loop.md" />

## Sources

- https://langfuse.com/blog/joining-clickhouse — Langfuse acquired by ClickHouse, open-source core unchanged, January 2026, accessed 2026-11-22
- https://clickhouse.com/blog/clickhouse-acquires-langfuse-open-source-llm-observability — core features remain MIT-licensed and self-hostable, accessed 2026-11-22
- https://github.com/Arize-ai/phoenix — Phoenix licensed under Elastic License 2.0, built on OpenTelemetry and OpenInference, accessed 2026-11-22
- https://ir.dynatrace.com/news-events/press-releases/detail/435/dynatrace-to-acquire-ai-observability-leader-arize — acquisition announced 2026-08-13, accessed 2026-11-22
- https://www.developer-tech.com/news/dynatrace-completes-arize-acquisition-for-ai-observability/ — acquisition completed, Phoenix and AX to continue, October 2026, accessed 2026-11-22
- https://mlflow.org/docs/latest/genai/tracing/opentelemetry/genai-semconv/ — MLflow `gen_ai.*` OTLP ingest and export, accessed 2026-11-22
- https://www.linuxfoundation.org/press/press-release/the-mlflow-project-joins-linux-foundation — MLflow a Linux Foundation project under Apache 2.0 (2020-06-25), accessed 2026-11-22
