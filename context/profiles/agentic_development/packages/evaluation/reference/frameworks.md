# Evaluation Framework Positioning

The three LLM-evaluation framework roles — CI-native gating, adversarial matrix testing, RAG-focused metrics — positioned as complements matched to failure modes.

No single evaluation framework covers every failure mode of a
non-deterministic system. The mature posture is to run complementary
framework roles in one eval tree, each selected for the class of failure it
detects — not to rank brands and pick a winner.

## The three roles

### CI-native metric gating — DeepEval

DeepEval integrates LLM-as-judge metrics with idiomatic pytest-style
fixtures, so evaluation suites run through the same test invocation and CI
gating as the rest of the repository (`deepeval test run`). Its role is
**regression prevention**: every behavior-affecting change passes its metric
gates before merge. Use it as the backbone the other roles plug into. Its
current line (4.x, Apache-2.0) covers custom-criteria judges (G-Eval and
graph-shaped DAG metrics), agent metrics over traces (task completion, tool
correctness, argument correctness, plan adherence, step efficiency),
multi-turn conversational metrics, and MCP-use metrics.

### Adversarial matrix testing — Promptfoo

Promptfoo excels at **matrix experiments**: the same prompt set swept across
model versions, parameters, and adversarial inputs, including jailbreak and
prompt-injection resistance. Its role is **red-teaming and boundary
mapping** — finding the input shapes where behavior degrades — and
config-change regression, where a prompt or model bump is swept against the
full matrix before rollout. Governance note: OpenAI announced its acquisition
of Promptfoo on 2026-03-09; the project states it remains open source under
the MIT license. Keep matrix configurations portable (plain YAML, provider-
neutral targets) so the role can move if that commitment changes.

### RAG-focused metric layer — RAGAS

RAGAS is a first-class complement tuned for retrieval-augmented and agentic
correctness: faithfulness, answer relevancy, contextual recall/precision, and
topic/goal adherence, plus tool-call accuracy and other agent metrics. Its
role is **retrieval and generation correctness**. The current line is 0.4.x
(0.4.4, November 2026) — there is no 1.0 yet. Use its collections API
(`ragas.metrics.collections`) and the `@experiment` runner: the legacy
`ragas.metrics` classes are deprecated in 0.4 and scheduled for removal in
1.0, `evaluate()` is deprecated, and some names changed (for example context
precision split into `ContextPrecisionWithReference` and
`ContextPrecisionWithoutReference`). New suites import only collections
metrics.

## Choosing is matching, not ranking

Map each gate you need to the role that detects its failure mode:

| Failure mode | Owning role |
|---|---|
| Behavior regression after a change | CI-native gating |
| Jailbreak / injection / unsafe output | Adversarial matrix |
| Prompt or model parameter drift | Adversarial matrix |
| Grounding, hallucination, retrieval quality | RAG-focused metric layer |
| Multi-turn goal drift | RAG-focused metric layer |
| Wrong tool, wrong arguments, wasted steps | CI-native gating over agent traces |
| Agent claims success the environment does not show | Outcome graders in the agent harness (see the agent-evals reference) |

All three write into the same dataset and observation formats so gates,
datasets, and the production-trace feedback loop stay shared infrastructure.

## Maturity posture

All three roles are current, actively maintained choices as of late 2026.
The deliberate story is the *composition*: a stack whose members are chosen
because each addresses a failure mode the others cannot see — CI gating for
known regressions, matrices for unknown ones, and a metric layer for
retrieval and grounding correctness. Pin each framework's version in the eval
tree: judge prompts and metric definitions change between releases, and an
unpinned bump silently moves every threshold built on them. Apply the same
license-and-governance grid the platforms reference uses — two of the three
reference tools changed ownership in 2026 or sit under single-vendor
control.

## Related

- <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::evaluation@agentic_development/reference/gates.md" />
- <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::evaluation@agentic_development/reference/datasets.md" />
- <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::evaluation@agentic_development/reference/feedback_loop.md" />
- <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::evaluation@agentic_development/reference/agent_evals.md" />

## Sources

- https://pypi.org/project/deepeval/ — DeepEval 4.3.4 (2026-11-11), Apache-2.0, agent, multi-turn, RAG and MCP metric families, `deepeval test run`, accessed 2026-11-22
- https://openai.com/index/openai-to-acquire-promptfoo/ — OpenAI to acquire Promptfoo, 2026-03-09, accessed 2026-11-22
- https://github.com/promptfoo/promptfoo — "remains open source and MIT licensed", accessed 2026-11-22
- https://github.com/explodinggradients/ragas/releases — RAGAS 0.4.4 latest, collections migration across 0.3.8–0.4.2, legacy metrics deprecated, accessed 2026-11-22
- https://docs.ragas.io/en/latest/howtos/migrations/migrate_from_v03_to_v04/ — collections import path, `@experiment`, deprecated `evaluate()`, renamed metrics, accessed 2026-11-22
