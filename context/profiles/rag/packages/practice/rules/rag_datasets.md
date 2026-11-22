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

# RAG Datasets and Architecture

Seed RAG golden datasets from production traces: a flagged failure — wrong
answer, missed context, hallucinated detail — becomes a labeled regression
case. Include adversarial cases the corpus cannot answer, where correct
behavior is refusal rather than invention. Dataset lifecycle mechanics
(versioning, promotion) are the `agentic_development` profile's; this
profile owns RAG-specific construction.

Architecturally, keep chunking, retrieval, and generation as separated,
independently testable stages, and treat chunking strategy as a decision
evaluated against the contextual metrics — never an unmeasured default. The
same holds for hybrid lexical + dense retrieval, chunk contextualization,
reranking, and late interaction: each is adopted on its measured delta. Check
first whether the corpus fits the context window outright.

- construction sources, labeling, and adversarial RAG cases — <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::practice@rag/reference/dataset_construction.md" />
- stage separation, chunking criteria, retrieval patterns, and context assembly signals — <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::practice@rag/reference/architecture.md" />
