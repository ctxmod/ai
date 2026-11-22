# Evaluation-First Development

Evaluation-first development for non-deterministic systems — define gates before building, split statistical from deterministic verification, and tier suites by cost.

A non-deterministic system cannot be asserted correct from deterministic
tests alone. Unit tests verify the deterministic surface around the model —
parsing, plumbing, state transitions — but the model's behavior itself has
no passing test, only distributions. Evaluation is therefore not a QA
afterthought bolted on before release; it is the primary quality instrument
for everything downstream of a prompt.

## The ordering discipline

Define the evaluation criteria and gates **before** building the pipeline:
which behaviors matter, how each is measured, what threshold constitutes
correct. Building first and evaluating after discovers that the system was
optimized for nothing in particular. The eval tree and the implementation
grow together, with the gates leading.

## The verification split

Non-deterministic behavior is verified statistically in the eval tree;
deterministic behavior — tool implementations, parsers, aggregations — is
verified by external black-box test tools. Source packages hold no internal
unit tests: verification lives outside the code it verifies, as executable
tools against built artifacts. The split matters for AI systems because the
two halves fail differently — deterministic code fails loudly and
reproducibly, while model behavior fails softly and statistically, and each
needs its own verification form.

## Done means gated

A behavior change to an agent, prompt, or retrieval path is complete when its
eval gates pass on the pinned dataset version. "It worked when I tried it" is
an anecdote, not evidence; a change without a gate result is not done.

## Tier the suites by cost

Evaluation costs tokens and wall time. Tier the suites so the cost lands
where the signal is:

- **Smoke gate** — a small, fast subset run on every change.
- **Full suite** — the complete golden set run on merge or release
  candidates.
- **Adversarial matrix** — swept on prompt/model changes and on a cadence,
  not every commit.

Each tier is an explicit configuration registered with the suite runner;
nothing runs implicitly.

## Related

- <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::evaluation@agentic_development/reference/gates.md" />
- <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::evaluation@agentic_development/reference/datasets.md" />
- <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::evaluation@agentic_development/reference/observation_not_verdict.md" />
