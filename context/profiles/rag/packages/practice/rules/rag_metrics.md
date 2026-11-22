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

# RAG Metrics

Evaluate retrieval-augmented generation with the retrieval metric layer:
faithfulness (claims grounded in retrieved context), answer relevancy, and
contextual recall/precision for the retrieval stage; topic and goal
adherence for multi-turn use. Where cases carry chunk-id labels, prefer the
deterministic label-based retrieval metrics (recall@k at the k the pipeline
passes on, with MRR and nDCG@k as explanations) over judged ones. Use the
RAGAS 0.4.x collections API (`ragas.metrics.collections`) — the legacy metric
classes are deprecated and slated for removal in 1.0, and names do not map
one-for-one.

Hallucination gating is a faithfulness threshold applied as a statistical
gate (for example, ≥ 0.95 over the pinned golden set). The gate mechanics —
sample size, distribution tracking, judge calibration — belong to the
`agentic_development` profile
(<ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::evaluation@agentic_development/reference/gates.md" />); this
profile owns only the RAG-specific metrics and their thresholds.

- the full metric set, label-based retrieval metrics, API notes, and failing-gate localization — <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::practice@rag/reference/metrics.md" />
- evaluating the retrieval stage separately from generation — <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::practice@rag/reference/retrieval_eval.md" />
