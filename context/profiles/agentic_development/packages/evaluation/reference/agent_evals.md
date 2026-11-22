# Evaluating Agents

Agent and trajectory evaluation — tasks, trials, graders, transcripts and outcomes; grading the environment's end state over the agent's claims; isolated trials; and reading transcripts before trusting a score.

An agent is a multi-step, tool-using subject: it reads state, calls tools,
changes the world, and reports what it did. A score over its final message
alone measures its narration. Agent evaluation grades what happened.

## The vocabulary

- **Task** — one test case: defined inputs, an environment, and success
  criteria.
- **Trial** — one attempt at a task. Agents are stochastic, so a task runs
  several trials, and the gate statistics (including pass^k) are computed
  over them.
- **Grader** — logic that scores one aspect of a trial. A task usually has
  several, each answering one named question.
- **Transcript** (trace, trajectory) — the complete record of a trial:
  messages, reasoning where available, tool calls and results.
- **Outcome** — the final state of the environment after the trial.

## Grade the outcome, then the path

The primary grader checks the **outcome**: the reservation exists in the
database, the file compiles, the ticket is in the right state. An agent that
says it booked the flight and did not is a failure no transcript reading
should rescue. Outcome grading is usually deterministic code — the cheapest,
most reproducible grader there is.

Transcript graders come second and answer path questions the outcome
cannot: was the right tool selected with valid arguments, were destructive
actions confirmed (an A2A `input-required` pause, an MCP multi-round-trip
confirmation), were steps wasted, did the agent stay within its permissions.
Grade the path for properties that matter — safety, cost, policy — and not
for matching one golden sequence: agents legitimately reach the same outcome
by different routes, and an exact-trajectory match penalizes valid
solutions.

| Grader kind | Strengths | Weaknesses | Use for |
|---|---|---|---|
| Code | fast, cheap, objective, reproducible | brittle to valid variation, weak on nuance | outcome state, schema-valid arguments, forbidden calls |
| Model (LLM judge) | flexible, handles open-ended output | non-deterministic, costlier, needs calibration | plan quality, explanation faithfulness, tone |
| Human | highest fidelity | slow, expensive | calibrating model graders, reviewing failures |

Model graders follow the calibration discipline in the gates reference.

## Isolate every trial

Each trial starts from a **clean environment**: fresh state, no files or
caches from earlier trials, no shared rate-limit or resource exhaustion.
Shared state produces correlated failures that come from the harness rather
than the agent, and can inflate scores — an agent that reads artifacts left
by an earlier trial is grading itself against the answer key. Environment
setup and teardown are part of the harness, versioned with the tasks.

## Read the transcripts

A score is a claim about graders as much as about the agent. Before trusting
a suite, read a sample of transcripts from both passing and failing trials:
a failure should have a clear, fair reason, and a pass should not be a
grader that accepted an invalid solution. Grader bugs found this way are
fixed in the grader, with the affected results recomputed from the stored
observations — the observation-not-verdict discipline is what makes that
recomputation possible.

## Related

- <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::evaluation@agentic_development/reference/gates.md" />
- <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::evaluation@agentic_development/reference/observation_not_verdict.md" />
- <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::evaluation@agentic_development/reference/frameworks.md" />
- <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::telemetry@agentic_development/reference/otel_genai.md" />

## Sources

- https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents — task, trial, grader, transcript and outcome definitions; grader types and tradeoffs; clean-environment trials; reading transcripts, 2026-01-09, accessed 2026-11-22
- https://pypi.org/project/deepeval/ — agent metric families over traces (task completion, tool correctness, argument correctness, plan adherence, step efficiency), DeepEval 4.3.4, accessed 2026-11-22
