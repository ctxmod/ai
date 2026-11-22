# Statistical Gates, Not Boolean Assertions

Gate non-deterministic behavior with statistical thresholds over golden datasets — LLM-as-judge calibration, distribution tracking, and the one Boolean exception.

An LLM output is a sample from a distribution, not the result of a function.
A single passing run proves nothing; a single failing run proves little.
Boolean assertions — "this test passed" — answer the wrong question for
non-deterministic systems. The correct unit of verification is the
**statistical gate**: a metric, a threshold, and a sample size, evaluated
over a pinned golden-dataset version.

## Anatomy of a gate

Every gate names four things:

1. **Metric** — what is measured (faithfulness, task-completion rate,
   routing precision, latency).
2. **Threshold** — the bound that must hold (for example, faithfulness
   ≥ 0.95; routing precision ≥ 0.98 over 50 scenarios).
3. **Sample** — the dataset version and case count the metric is computed
   over. A threshold without a sample size is decoration.
4. **Regression semantics** — what counts as a regression: statistically
   significant degradation on the pinned dataset, not any single failed
   case. Track score distributions across runs, not just point estimates.

## LLM-as-judge is itself a model

Metrics computed by an LLM judge inherit every failure mode of a model.
Calibrate the judge before trusting its gates: pin the judge model and rubric,
check judge agreement against human-labeled cases, and control for position
and verbosity bias. A judge whose scores drift between runs invalidates every
threshold built on it.

## The one Boolean exception

A categorical safety invariant gates on exact rejection, not a rate: an
adversarial matrix of dangerous inputs must be rejected **100%** of the time,
across every activated skill and every matrix cell. This is the only place a
Boolean assertion is correct — the property is deterministic by policy even
though the system producing it is not.

## Gates are thresholds on observations

Gates consume structured observations produced by evaluation runs; they never
produce them. The observation record — a named question about a named subject
at a named presentation — is the durable artifact, and the gate verdict is
derived from it, recomputable at any time. See
`reference/principles/observation-not-verdict.md` in this profile.

## Related

- <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::evaluation@agentic_development/reference/frameworks.md" />
- <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::evaluation@agentic_development/reference/datasets.md" />
- <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::evaluation@agentic_development/reference/observation_not_verdict.md" />
