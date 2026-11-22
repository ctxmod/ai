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

# Evaluation Pipeline Authoring

Pick evaluation tooling by failure mode, not brand ranking: a CI-native
metric-gating framework (DeepEval) for regression suites wired into standard
CI; an adversarial matrix runner (Promptfoo) for jailbreak, injection, and
config-change regression; a RAG-focused metric layer (RAGAS 0.4.x, via its
`ragas.metrics.collections` API — the legacy metric classes are deprecated)
for retrieval and generation correctness. The three roles compose in one eval
tree; each gate is owned by the role that detects its failure. Pin every
framework and judge version in the eval tree. Agents are graded on the
environment's end state first and their trajectory second, each trial in a
clean environment.

Golden datasets are versioned files in the eval tree that grow from
production traces: a user-flagged failure becomes a permanent regression
case before the next build ships.

- framework role positioning and the maturity posture — <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::evaluation@agentic_development/reference/frameworks.md" />
- agent evaluation: tasks, trials, outcome and transcript graders — <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::evaluation@agentic_development/reference/agent_evals.md" />
- golden dataset lifecycle: case shape, promotion, pruning, versioning — <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::evaluation@agentic_development/reference/datasets.md" />
- the trace → dataset → eval loop and its selection signals — <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::evaluation@agentic_development/reference/feedback_loop.md" />
