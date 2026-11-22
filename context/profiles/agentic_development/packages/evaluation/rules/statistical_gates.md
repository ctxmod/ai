---
paths:
  - "eval/*"
  - "eval/*/*"
  - "eval/*/*/*"
  - "eval/*/*/*/*"
  - "*/eval/*"
  - "*/eval/*/*"
  - "*/eval/*/*/*"
  - "*/*/eval/*"
  - "*/*/eval/*/*"
  - "*/*/*/eval/*"
---

# Statistical Gates

LLM outputs are distributions, not functions: a single passing run proves
nothing and a single failing run proves little. Gate every non-deterministic
behavior with a statistical gate — a metric, a threshold, and a sample size —
evaluated over a pinned golden-dataset version. Regression means statistically
significant degradation on that dataset, not any one failed case.

The one Boolean exception is a categorical safety invariant: an adversarial
matrix that must be rejected 100% of the time gates on exact rejection, not
on a rate.

Evaluation runs record structured observations — a named question about a
named subject at a named presentation — and verdicts are derived from them,
never hand-written.

- gate anatomy, LLM-as-judge calibration, and the safety exception — <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::evaluation@agentic_development/reference/gates.md" />
- the observation-not-verdict philosophy and derivable result layouts — <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::evaluation@agentic_development/reference/observation_not_verdict.md" />
- why evaluation precedes implementation, and how suites tier by cost — <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::evaluation@agentic_development/reference/evaluation_first.md" />
