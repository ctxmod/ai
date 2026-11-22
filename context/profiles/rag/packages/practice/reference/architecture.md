# RAG Architecture Criteria

RAG architecture criteria — whether retrieval is needed at all, chunking, hybrid retrieval, chunk contextualization, late interaction, reranking, and context assembly quality signals, each independently testable.

A RAG pipeline is a chain of separable stages. The architecture review asks
one question of each: is this stage a deliberate, evaluated decision, or a
default that was never measured?

## The stage chain and its separation rule

```
corpus → chunking (+ contextualization) → retrieval (lexical ∥ dense → fusion) → rerank → context assembly → generation
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

## Before RAG: does the corpus fit?

If the whole knowledge base fits comfortably in the model's context (on the
order of 200,000 tokens or less), placing it in the prompt — with prompt
caching to amortize cost — can beat a retrieval pipeline outright and removes
a whole class of recall failures. Retrieval earns its complexity when the
corpus outgrows the window or changes faster than a cached prompt can.

## Retrieval patterns worth evaluating

The current baseline is not a single dense index. Each pattern below is a
candidate stage, adopted when it moves the retrieval metrics on the pinned
dataset:

- **Hybrid retrieval** — a lexical index (BM25) beside the dense one, their
  rankings fused, typically by **reciprocal rank fusion** (each document
  scores the sum of 1/(k + rank) over the lists it appears in; the original
  paper's k = 60 was near-optimal and not critical). Lexical
  retrieval catches exact identifiers, codes, and rare terms that embeddings
  blur; fusion needs no score calibration between the two.
- **Chunk contextualization** — prepend a short, model-generated description
  of where a chunk sits in its document before embedding and before lexical
  indexing, so a chunk stripped of its section still retrieves. The published
  measurement cut top-20 retrieval failures by 35% with contextual embeddings,
  49% adding contextual BM25, and 67% adding reranking; the indexing cost is
  one model call per chunk, which prompt caching keeps small.
- **Late interaction** — token-level multi-vector retrieval (the ColBERT
  family) for corpora where single-vector embeddings lose precision, and its
  vision-language form (ColPali) for visually rich documents indexed as page
  images instead of extracted text. It trades index size for precision; size
  it before adopting it.

## Reranking

A reranking stage — usually a cross-encoder scoring each query–chunk pair —
buys contextual precision at latency cost. The working shape is to
over-retrieve and rerank down (for example, around 150 candidates reranked to
the 20 that enter the window). Its justification is the measured precision
delta with and without, on the pinned dataset — not vendor claims.

## Context assembly quality signals

What actually enters the prompt window is a stage of its own. Signals to
instrument and review: relevance of assembled context (precision of the
final window, not just of raw retrieval), ordering (evidence position
affects groundedness), deduplication (near-duplicate chunks waste budget),
and truncation behavior (what gets dropped when the budget overflows —
silently truncating the most relevant chunk is the classic silent failure).

## Review checklist

For any RAG pipeline, every row should have a measured answer: whether the
corpus needs retrieval at all; chunking strategy and its evidence;
retrieval/generation separation testable; hybrid and contextualization
deltas; rerank delta; assembly signals exported as spans; refusal behavior for
unanswerable queries; failure localization via the metric pair (see the
metrics and retrieval-eval references). Datasets feeding these evaluations
are built per the dataset-construction reference.

## Related

- <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::practice@rag/reference/metrics.md" />
- <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::practice@rag/reference/retrieval_eval.md" />
- <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::practice@rag/reference/dataset_construction.md" />

## Sources

- https://www.anthropic.com/news/contextual-retrieval — contextual embeddings and contextual BM25, hybrid retrieval, rerank 150→20, failure-rate reductions, the ~200,000-token long-context alternative, 2024-09-19, accessed 2026-11-22
- http://cormack.uwaterloo.ca/cormacksigir09-rrf.pdf — Cormack, Clarke, Büttcher, "Reciprocal Rank Fusion Outperforms Condorcet and Individual Rank Learning Methods", SIGIR 2009 (RRF formula, k = 60), accessed 2026-11-22
- https://arxiv.org/abs/2004.12832 — Khattab and Zaharia, "ColBERT: Efficient and Effective Passage Search via Contextualized Late Interaction over BERT", 2020, accessed 2026-11-22
- https://arxiv.org/abs/2407.01449 — Faysse et al., "ColPali: Efficient Document Retrieval with Vision Language Models", 2024 (revised 2025-02-28), accessed 2026-11-22
