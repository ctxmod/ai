# RAG Architecture Criteria

RAG architecture criteria — chunking, retrieval/generation separation, reranking, and context assembly quality signals, each independently testable.

A RAG pipeline is a chain of separable stages. The architecture review asks
one question of each: is this stage a deliberate, evaluated decision, or a
default that was never measured?

## The stage chain and its separation rule

```
corpus → chunking → retrieval → rerank (optional) → context assembly → generation
```

Each stage is independently testable and independently swappable.
**Retrieval/generation separation is the load-bearing rule**: the retrieval
stage is evaluated against contextual recall/precision on labeled cases;
the generation stage against faithfulness and answer relevancy. Swapping one
must not force re-evaluating the other from scratch — if it would, the
boundary is fake.

## Chunking criteria

- Chunking strategy is an **evaluated decision**: size, overlap, and
  boundary policy are swept against contextual recall/precision, not set to
  library defaults and forgotten.
- Where the corpus has structure — sections, records, tables — boundary
  semantics beat fixed character windows; keep structure metadata attached
  to chunks so filters can use it.
- Corpus migrations re-open the chunking decision; chunk identity is part
  of the dataset's expected-context labeling.

## Reranking

A reranking stage buys contextual precision at latency cost. Its
justification is the measured precision delta with and without, on the
pinned dataset — not vendor claims.

## Context assembly quality signals

What actually enters the prompt window is a stage of its own. Signals to
instrument and review: relevance of assembled context (precision of the
final window, not just of raw retrieval), ordering (evidence position
affects groundedness), deduplication (near-duplicate chunks waste budget),
and truncation behavior (what gets dropped when the budget overflows —
silently truncating the most relevant chunk is the classic silent failure).

## Review checklist

For any RAG pipeline, every row should have a measured answer: chunking
strategy and its evidence; retrieval/generation separation testable; rerank
delta; assembly signals exported as spans; refusal behavior for
unanswerable queries; failure localization via the metric pair (see the
metrics and retrieval-eval references). Datasets feeding these evaluations
are built per the dataset-construction reference.

## Related

- <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::practice@rag/reference/metrics.md" />
- <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::practice@rag/reference/retrieval_eval.md" />
- <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::practice@rag/reference/dataset_construction.md" />
