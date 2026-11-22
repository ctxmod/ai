# The RAG Metric Layer

The RAG metric layer — faithfulness, answer relevancy, contextual recall/precision, topic/goal adherence, noise sensitivity, and label-based ranking metrics — via the RAGAS collections API, with faithfulness thresholds as hallucination gates.

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
| **Noise sensitivity** | Does irrelevant retrieved context pull the answer wrong? | Distractor chunks overriding the relevant ones |

Two families compute these. **Reference-based** metrics compare against
labels — expected chunks, a reference answer — and need the labeled dataset;
**reference-free** metrics judge the answer against the retrieved context
alone and run on unlabeled production traces. Gate on reference-based
metrics over the golden set; use reference-free ones to watch production and
to select traces for labeling.

## Label-based retrieval metrics

When cases carry expected-context labels (chunk or document ids), the
retrieval stage has deterministic, judge-free metrics from information
retrieval:

| Metric | Measures |
|---|---|
| **Recall@k** | share of the expected chunks that appear in the top k |
| **Precision@k** | share of the top k that are expected chunks |
| **MRR** | how early the first expected chunk appears (mean reciprocal rank) |
| **nDCG@k** | graded relevance with a position discount, normalized to the ideal ranking |

They cost nothing per run, never drift, and need no calibration, so they are
the first choice for retrieval gates whenever labels exist; LLM-judged
contextual recall and precision cover the cases where labels are claims
rather than chunk ids. Pick k to match what context assembly actually passes
to the model — recall@20 is meaningless for a pipeline that keeps five
chunks. RAGAS exposes the id-based form directly (for example an id-based
context-precision variant), and the formulas are simple enough to compute in
the eval tree.

## Modern APIs

Use the collections API of the metric layer (RAGAS 0.4.x,
`ragas.metrics.collections`). The legacy `ragas.metrics` classes are
deprecated in 0.4 and scheduled for removal in 1.0; new suites must not import
them. Class names do not map one-for-one between the two APIs — context
precision alone has reference-based, reference-free, and id-based variants —
so migrations follow the project's migration table and re-baseline the
affected thresholds, because a renamed metric may also be a changed judge
prompt.

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

## Sources

- https://docs.ragas.io/en/latest/concepts/metrics/available_metrics/ — current RAG metric set (context precision and recall, noise sensitivity, response relevancy, faithfulness) and agent metrics, accessed 2026-11-22
- https://docs.ragas.io/en/latest/concepts/metrics/available_metrics/context_precision/ — context precision as rank-weighted mean of precision@k; LLM, non-LLM, and id-based variants; legacy API removal in 1.0, accessed 2026-11-22
- https://github.com/explodinggradients/ragas/releases — RAGAS 0.4.4 latest release, accessed 2026-11-22
- https://www.anthropic.com/news/contextual-retrieval — label-based recall@20 (reported as 1 − recall@20) as the retrieval metric for comparing pipelines, 2024-09-19, accessed 2026-11-22
