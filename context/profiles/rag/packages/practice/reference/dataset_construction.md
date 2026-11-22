# Golden Dataset Construction from Production Traces

Building RAG golden datasets from production traces — user-flagged failures first, labeled expected context, and adversarial unanswerable cases.

A RAG golden dataset is only as honest as its sources. Hand-authored
questions over a clean corpus measure the system's best case; production
traces measure what users actually hit. Build from both, weighted toward
production.

## Sources and selection

Each trace yields the raw material of a case: the question asked, the chunks
retrieved, the answer generated, and any user reaction. Selection priority:

1. **User-flagged failures** — wrong answer, missed context, hallucinated
   detail. An explicit human judgment is the highest-value seed; every flag
   should become a case unless it duplicates one.
2. **Low-scoring traces** — judge or heuristic scores below threshold.
   Noisier than flags, but they surface failures users did not bother to
   report.
3. **Curated seeds** — hand-authored cases covering behaviors production
   has not exercised yet.

## Labeling

Labels are properties, not transcripts:

- which chunks (or which corpus regions) **should** be recalled — the
  expected-context labels retrieval evaluation depends on;
- what claims the answer must ground on, and what it must not invent;
- the expected refusal when the corpus cannot answer.

Expected-context labels are the expensive part; harvest them from the
trace's own retrieved chunks and correct, rather than authoring from
scratch.

## Adversarial RAG cases

Maintain a separate adversarial set with two shapes:

- **Unanswerable from context** — the correct behavior is a refusal or a
  "not in the material" answer; an answer is a hallucination and fails.
- **Unfaithful-answer traps** — questions whose plausible answer contradicts
  the retrieved evidence; faithfulness must catch the contradiction.

These gate on the agentic-development profile's safety semantics — exact
rejection for the refusal invariant, statistical gates for the rest.

## Hygiene and ownership

Version RAG cases alongside the rest of the eval tree, tag provenance
(trace, flag, hand-authored), dedup on question + corpus version, and review
for staleness when the corpus changes materially — a corpus migration
re-opens retrieval labels wholesale. Lifecycle mechanics (promotion rules,
immutability of gated versions) are the `agentic_development` profile's;
this profile owns the RAG-specific construction.

## Related

- <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::practice@rag/reference/metrics.md" />
- <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::practice@rag/reference/retrieval_eval.md" />
- <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::practice@rag/reference/architecture.md" />
