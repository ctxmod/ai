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
adherence for multi-turn use. Use the modern collections-style APIs — the
legacy metric class names are deprecated in the current major version.

Hallucination gating is a faithfulness threshold applied as a statistical
gate (for example, ≥ 0.95 over the pinned golden set). The gate mechanics —
sample size, distribution tracking, judge calibration — belong to the
`agentic_development` profile
(<ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::evaluation@agentic_development/reference/gates.md" />); this
profile owns only the RAG-specific metrics and their thresholds.

- the full metric set, modern-API notes, and failing-gate localization — <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::practice@rag/reference/metrics.md" />
- evaluating the retrieval stage separately from generation — <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::practice@rag/reference/retrieval_eval.md" />
