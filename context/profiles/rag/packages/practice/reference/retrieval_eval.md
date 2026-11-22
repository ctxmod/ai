# Retrieval Quality Evaluation

Evaluate the retrieval stage separately from generation — sweep chunking and ranking against contextual recall/precision, and use the metric pair to localize failures.

Retrieval and generation fail independently, so they are evaluated
independently. Folding them into one end-to-end score hides which stage
broke; separating them turns every failure into a localization.

## The diagnosis matrix

Read a faithfulness gate failure through contextual recall:

| Faithfulness | Contextual recall | Localized failure |
|---|---|---|
| fail | fail | **Retrieval** — the evidence never reached the window; the model invented around the gap |
| fail | pass | **Generation** — the evidence was retrieved and ignored or contradicted |
| pass | fail | Latent risk — right answers for wrong reasons; fix before it surfaces |

The third row matters most and is the one end-to-end scoring never shows:
a retrieval stage can degrade for weeks behind a generation stage that
compensates.

## What to sweep

Treat retrieval configuration as hypotheses tested by matrix experiments
against contextual recall and contextual precision:

- **Chunking** — size, overlap, and boundary strategy. Structure-aware
  boundaries (sections, records, paragraphs) usually beat fixed windows;
  measure rather than assume.
- **Top-k and window budget** — how much context enters the prompt, and
  what precision it carries.
- **Reranking** — with and without a reranking stage; the precision delta
  is its justification.
- **Embedding / index choice** — swapped behind the same suite, so
  candidates are comparable on the pinned dataset version.
- **Retriever composition** — dense only versus hybrid lexical + dense, and
  chunk contextualization on or off; each is a measured delta, not an
  assumed upgrade (see the architecture reference).

## Ranking quality signals

When cases carry chunk-id labels, the label-based ranking metrics —
recall@k, precision@k, MRR, nDCG@k (defined in the metrics reference) — are
deterministic and judge-free, so recall@k at the k the pipeline actually
passes on is a legitimate retrieval gate in its own right. The finer-grained
ranking measures (MRR, nDCG) stay observational: they do not gate by
themselves; they explain *why* recall or contextual precision moved. Where
labels are claims rather than chunk ids, the LLM-judged contextual metrics
remain the gate.

## Dataset coupling

Retrieval evaluation needs cases with labeled expected context — which
chunks should be recalled for which question. Those labels are part of
golden-dataset construction (see the dataset-construction reference), and
they are the reason production traces with their retrieved chunks are
valuable seeds: they arrive pre-paired.

## Related

- <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::practice@rag/reference/metrics.md" />
- <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::practice@rag/reference/dataset_construction.md" />
- <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::practice@rag/reference/architecture.md" />

## Sources

- https://www.anthropic.com/news/contextual-retrieval — retrieval configurations compared on recall@20 over labeled chunks (embeddings, hybrid with BM25, contextualization, reranking), 2024-09-19, accessed 2026-11-22
- https://docs.ragas.io/en/latest/concepts/metrics/available_metrics/context_precision/ — id-based and LLM-based context precision variants, accessed 2026-11-22
