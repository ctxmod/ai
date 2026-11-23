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

## The statistics a gate needs

A threshold is only meaningful next to its uncertainty. Treat the cases as a
sample from the space of inputs the system will meet, and report:

- **A confidence interval, not a bare mean.** Report the standard error with
  every score (95% interval ≈ mean ± 1.96 × SE). A gate passes when the
  bound it cares about clears the threshold, not when the point estimate
  does.
- **Clustered standard errors** when cases share a source (several questions
  over one document, several turns of one conversation): naive standard
  errors understate uncertainty, sometimes badly.
- **Paired comparisons** for regression gates. Old and new subjects run on
  the same cases, so compare per-case differences; pairing removes
  case-difficulty variance and detects smaller regressions with the same
  sample.
- **Repeated trials per case** for stochastic subjects, averaged per case
  before aggregating, to shrink within-case variance.
- **Power analysis before the dataset is sized.** Decide the smallest
  regression worth catching and compute how many cases (and trials per case)
  detect it at the chosen confidence. A gate whose sample cannot detect the
  regression it claims to guard is decoration with a threshold.

## Reliability is a different metric from capability

For agents and other multi-step subjects, distinguish:

- **pass@k** — the probability that at least one of k trials succeeds; it
  rises with k and fits tasks where one good attempt is enough (a human picks
  the best of several drafts).
- **pass^k** — the probability that all k trials succeed; it falls with k and
  is the metric for anything that runs unattended (75% per-trial success is
  about 42% over three trials).

Gate user-facing automation on pass^k at the k it will actually be exercised;
a single-trial success rate overstates reliability.

## Capability versus regression suites

Keep the two purposes apart. A **capability suite** asks what the system can
do and should start with a low pass rate, leaving room to measure
improvement. A **regression suite** asks whether it still does what it used
to and should sit near 100%. Cases graduate from capability to regression as
they become reliably passed; a capability suite that passes everything is
saturated and needs harder cases, because near saturation large gains look
like small score changes.

## LLM-as-judge is itself a model

Metrics computed by an LLM judge inherit every failure mode of a model.
Calibrate the judge before trusting its gates:

- **Pin the judge** — model identifier, version, rubric, and sampling
  parameters are part of the gate definition; changing any of them is a
  re-baseline, recorded like a dataset version change.
- **Measure it against humans** — hold out a human-labeled set and report
  the judge's agreement with it as true-positive and true-negative rates (or
  chance-corrected agreement), not a single accuracy figure; a judge that
  passes everything scores well on a mostly-passing set.
- **Control the known biases** — position bias (swap the order in pairwise
  comparisons and require agreement), verbosity bias (longer is not better),
  and self-preference (prefer a judge from a different model family than the
  subject).
- **One dimension per judge** — a clear, structured rubric per criterion,
  each graded by an isolated judge call that returns a rationale, rather than
  one call scoring every dimension; vague rubrics produce inconsistent
  verdicts. Give the judge a way out ("unknown" when the evidence is
  insufficient) instead of forcing a guess, and give multi-part tasks
  partial credit.
- **Prefer code where code suffices** — exact matches, schema checks, and
  environment-state assertions are cheaper, deterministic, and need no
  calibration; reserve judges for what only a judge can assess.

Re-check calibration on a cadence and whenever the subject's output
distribution shifts. A judge whose scores drift between runs invalidates
every threshold built on it.

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
<ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::evaluation@agentic_development/reference/observation_not_verdict.md" /> in this profile.

## Related

- <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::evaluation@agentic_development/reference/frameworks.md" />
- <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::evaluation@agentic_development/reference/datasets.md" />
- <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::evaluation@agentic_development/reference/agent_evals.md" />

## Sources

- https://www.anthropic.com/research/statistical-approach-to-model-evals — standard errors, clustered standard errors, resampling, paired differences, power analysis for model evals, accessed 2026-11-22
- https://arxiv.org/abs/2411.00640 — Miller, "Adding Error Bars to Evals: A Statistical Approach to Language Model Evaluations", 2024-11-01, accessed 2026-11-22
- https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents — pass@k versus pass^k, capability versus regression evals, saturation, grader calibration against humans, 2026-01-09, accessed 2026-11-22
- https://arxiv.org/abs/2306.05685 — Zheng et al., "Judging LLM-as-a-Judge with MT-Bench and Chatbot Arena" (position, verbosity, and self-enhancement bias), NeurIPS 2023, accessed 2026-11-22
