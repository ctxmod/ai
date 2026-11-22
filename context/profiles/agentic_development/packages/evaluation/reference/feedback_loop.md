# The Trace → Dataset → Eval Feedback Loop

The production-trace → dataset → eval feedback loop — user flags as the highest-signal selection mechanism, and the promotion rule that makes failures permanent regression cases.

An evaluation suite that only ever runs hand-authored cases measures the
system you imagined. The loop that keeps it measuring the system users
actually get runs continuously: production traces are captured by the
observability platform, selected failures become golden-dataset cases, and
the eval suite turns each one into a permanent regression gate.

## The loop

1. **Capture** — agent and tool spans stream to the observability platform
   (trace, inputs, outputs, tool calls, scores).
2. **Select** — the highest-signal selection mechanism is the **user flag**:
   an explicit human judgment that a specific interaction was wrong. Low
   judge scores and anomaly signals rank second; both are cheaper to collect
   and noisier than a flag.
3. **Promote** — a sync job pulls the flagged trace (session id, active
   configuration, user note) and commits it to the golden dataset with
   provenance tags. The failure becomes a permanent regression case
   **before the next build ships** — that is the loop's non-negotiable step.
4. **Gate** — the next evaluation run includes the new case; a fix is done
   when the case passes and stays passing.

## Worked patterns

### Skill-routing precision

Dynamic skill activation is evaluated as a routing gate: run the scenario
suite, and require that when a prompt calls for a specific capability, the
agent activates the matching skill with at least the gate threshold (for
example ≥ 98% over 50 scenarios). Routing failures found in production enter
the dataset through the same flag → promote → gate path.

### Faithfulness to provided state

When the system answers questions over state the caller provides (an
inventory, a document set, a user profile), gate on whether generated output
references only what the state contains. A faithfulness metric computed
against the provided payload detects invention; the threshold is set as any
other statistical gate.

## Tooling shape

The observability platform's dataset/experiment surface (see the platforms
reference) usually provides the pull side; a small sync job — scheduled or
invoked from the flag action — performs the commit. Keep the sync job in the
eval tree so the loop's mechanics are versioned with the datasets they feed.

## Related

- <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::evaluation@agentic_development/reference/datasets.md" />
- <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::evaluation@agentic_development/reference/gates.md" />
- <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::telemetry@agentic_development/reference/platforms.md" />
