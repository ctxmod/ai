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
gating as the rest of the repository. Its role is **regression prevention**:
every behavior-affecting change passes its metric gates before merge. Use it
as the backbone the other roles plug into.

### Adversarial matrix testing — Promptfoo

Promptfoo excels at **matrix experiments**: the same prompt set swept across
model versions, parameters, and adversarial inputs, including jailbreak and
prompt-injection resistance. Its role is **red-teaming and boundary
mapping** — finding the input shapes where behavior degrades — and
config-change regression, where a prompt or model bump is swept against the
full matrix before rollout.

### RAG-focused metric layer — RAGAS

RAGAS is a first-class complement tuned for retrieval-augmented and agentic
correctness: faithfulness, answer relevancy, contextual recall/precision, and
topic/goal adherence. It is not a legacy option — as of 2026 it ships
quickstarts and a richer metric set including agent metrics. Its role is
**retrieval and generation correctness**. Use its modern collections-style
API; the legacy metric class names (for example the old
`TopicAdherenceScore` shape) are deprecated for the current major version.

## Choosing is matching, not ranking

Map each gate you need to the role that detects its failure mode:

| Failure mode | Owning role |
|---|---|
| Behavior regression after a change | CI-native gating |
| Jailbreak / injection / unsafe output | Adversarial matrix |
| Prompt or model parameter drift | Adversarial matrix |
| Grounding, hallucination, retrieval quality | RAG-focused metric layer |
| Multi-turn goal drift | RAG-focused metric layer |

All three write into the same dataset and observation formats so gates,
datasets, and the production-trace feedback loop stay shared infrastructure.

## Maturity posture

All three roles are current, actively maintained choices in 2026. The
deliberate story is the *composition*: a stack whose members are chosen
because each addresses a failure mode the others cannot see — CI gating for
known regressions, matrices for unknown ones, and a metric layer for
retrieval and grounding correctness.

## Related

- <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::evaluation@agentic_development/reference/gates.md" />
- <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::evaluation@agentic_development/reference/datasets.md" />
- <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::evaluation@agentic_development/reference/feedback_loop.md" />
