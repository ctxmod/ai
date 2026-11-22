# Golden Dataset Lifecycle

The golden dataset lifecycle — seeding, growing from production traces, promotion rules, provenance tags, and adversarial sets.

A golden dataset is the pinned sample a statistical gate is computed over.
It lives as a versioned file in the eval tree (JSONL or YAML), committed like
code, so every gate result names the exact dataset version it was computed
against.

## Case shape

Each case carries at minimum:

- **Input** — the prompt, scenario, or request as presented to the system.
- **Expected properties** — what a correct response must satisfy: claims that
  must be grounded, tools that must be selected, state that must be
  referenced. Expectations are properties, not verbatim transcripts; the
  system is non-deterministic and exact-output matching is meaningless.
- **Provenance tag** — where the case came from: hand-curated seed,
  production trace, user flag, or adversarial construction.

## Lifecycle stages

1. **Seed** — hand-author the initial cases covering the core behaviors the
   system must exhibit. Small and high-signal beats large and random.
2. **Grow** — production traces and user flags are the primary growth
   source; see the feedback-loop reference for the selection mechanics.
3. **Promote** — a promoted case is permanent: it gates every future change.
   Promotion requires the failure to be understood (root cause identified),
   not just observed.
4. **Prune** — review periodically for staleness: cases tied to retired
   features, duplicated coverage, or expectations the product no longer
   holds. Removal is a recorded decision, not silent drift.

## Adversarial sets are separate files

Keep adversarial cases (injection attempts, jailbreaks, unsafe-input
matrices) in their own versioned files. They gate differently — exact
rejection rather than scored metrics — and they grow from red-team sessions
rather than production traffic.

## Versioning discipline

Datasets are immutable once a gate has been run against them: changes are new
versions, and a dataset version plus a subject version uniquely identify a
gate result. Re-baselining a threshold requires naming both the old and new
dataset versions in the change that re-baselines it.

## Related

- <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::evaluation@agentic_development/reference/gates.md" />
- <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::evaluation@agentic_development/reference/feedback_loop.md" />
- <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::evaluation@agentic_development/reference/frameworks.md" />
