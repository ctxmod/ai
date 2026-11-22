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

- **Langfuse** — the current default open platform: permissive-licensed,
  self-hostable, with native dataset/experiment/eval surface and
  OpenTelemetry-compatible ingest. Its agent-eval loop (traces + datasets +
  LLM-as-judge in one place) is the pattern the loop above describes.
- **Phoenix (Arize)** — capable and widely adopted, but **source-available
  under the Elastic License 2.0, not OSI open source**, and Arize has been
  mid-acquisition by Dynatrace — long-term governance of the open project
  is genuinely unclear. OpenInference itself remains backend-neutral; the
  governance question is about the platform, not the convention.
- **Hosted commercial alternatives** — LangSmith, Braintrust, and lighter
  entries like Laminar trade self-hosting for operational convenience.

Evaluate every platform on the same grid: **license** (OSS vs
source-available), **governance trajectory** (who owns it, who is buying
whom), **self-host capability** (can traces stay in-cluster),
**ingest compatibility** (standard OTLP and the OpenInference layer, not a
proprietary SDK requirement), and **dataset/experiment API surface**
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
