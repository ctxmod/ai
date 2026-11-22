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
evaluated over a pinned golden-dataset version, reported with its confidence
interval. Regression means statistically significant degradation on that
dataset — judged by a paired comparison over the same cases — not any one
failed case. Size the dataset by power analysis for the smallest regression
worth catching, and gate unattended agents on pass^k, not single-trial
success. Pin and calibrate LLM judges against human labels before their
scores gate anything.

The one Boolean exception is a categorical safety invariant: an adversarial
matrix that must be rejected 100% of the time gates on exact rejection, not
on a rate.

Evaluation runs record structured observations — a named question about a
named subject at a named presentation — and verdicts are derived from them,
never hand-written.

- gate anatomy, gate statistics, pass@k versus pass^k, LLM-as-judge calibration, and the safety exception — <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::evaluation@agentic_development/reference/gates.md" />
- agent evaluation: outcome-first graders, isolated trials, transcript review — <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::evaluation@agentic_development/reference/agent_evals.md" />
- the observation-not-verdict philosophy and derivable result layouts — <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::evaluation@agentic_development/reference/observation_not_verdict.md" />
- why evaluation precedes implementation, and how suites tier by cost — <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::evaluation@agentic_development/reference/evaluation_first.md" />
