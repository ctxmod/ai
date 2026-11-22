# The RAG Metric Layer

The RAG metric layer — faithfulness, answer relevancy, contextual recall/precision, topic/goal adherence — via modern collections-style APIs, with faithfulness thresholds as hallucination gates.

Retrieval-augmented generation needs metrics that separate "the answer is
grounded" from "the answer is good" from "the right context was found". The
metric layer covers all three, and each maps to a distinct failure mode.

## The metric set

| Metric | Question it answers | Failure it catches |
|---|---|---|
| **Faithfulness** | Are all claims grounded in the retrieved context? | Hallucination — invention beyond the evidence |
| **Answer relevancy** | Does the answer address the question asked? | Evasion, drift, padded non-answers |
| **Contextual recall** | Does the retrieved context cover the ground truth? | Missed evidence — chunking or top-k failures |
| **Contextual precision** | Do relevant chunks rank ahead of irrelevant ones? | Ranking noise diluting the window |
| **Topic / goal adherence** | Does a multi-turn exchange stay on the user's goal? | Conversational drift across turns |

## Modern APIs

Use the modern collections-style APIs of the metric layer (RAGAS in its
current major version). The legacy metric class names are deprecated for
v1.0-era interfaces; new suites must not import them, and migrations replace
them one-for-one with the collections equivalents.

## Hallucination gating

Hallucination control is a **faithfulness threshold applied as a statistical
gate**: for example, faithfulness ≥ 0.95 over the pinned golden-dataset
version, with regression semantics and sample-size discipline. The gate
mechanics themselves — sample size, distribution tracking, judge
calibration — are owned by the `agentic_development` profile; see
<ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::evaluation@agentic_development/reference/gates.md" />. This
profile owns the RAG-specific metrics and their thresholds, never the gate
machinery.

When the answer draws on caller-provided state rather than a retrieved
corpus, the same faithfulness metric gates against that payload — the
general pattern is the agentic-development profile's faithfulness-to-
provided-state example.

## Reading a failing gate

A faithfulness failure is a symptom, not a diagnosis. Route it:
poor contextual recall alongside it points at retrieval (see the
retrieval-eval reference); good recall points at generation. The metric
layer localizes the failure; the architecture reference says what to change.

## Related

- <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::practice@rag/reference/retrieval_eval.md" />
- <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::practice@rag/reference/dataset_construction.md" />
- <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::practice@rag/reference/architecture.md" />
