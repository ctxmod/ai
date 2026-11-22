# Observation, Not Verdict

Evaluation runs record structured observations — a named question about a named subject at a named presentation — stored at derivable paths; verdicts are derived, never hand-written.

Every evaluation run answers a named question about a named subject at a
named presentation. Keeping those three explicit is what separates an
evaluation platform from a folder of test scripts.

- **Named question** — what is being asked ("does the agent route
  consolidation requests to the right skill?"), not a vague "does it work".
- **Named subject** — exactly what is under test: pipeline version, prompt
  version, model identifier, skill set.
- **Named presentation** — the input shape and conditions: dataset version,
  sampling temperature, transport, adversarial or clean.

## Observations are the durable record

A run produces structured observations — per-case scores, judge rationales,
latencies, tool traces — written to a path **derivable before the artifact
exists**: a deterministic layout keyed by subject version, dataset version,
and timestamp, not a browser-exported file named whatever the tool chose.
Derivable paths mean any consumer can locate a run's evidence without
discovering it, and retention can be reasoned about mechanically.

## Verdicts are derived

A verdict — pass, fail, regressed — is computed from observations by a gate.
It is never hand-written, never stored as the primary record, and always
recomputable: given the observation set and the gate definition, anyone can
re-derive the verdict. When a verdict and its evidence disagree, the verdict
is wrong by definition.

## Evidence before verdicts in review

Review culture follows the same order: read the observations first
(distributions, failure cases, judge rationales), then the derived verdicts.
A green aggregate over a degraded distribution is a verdict without evidence
— the most dangerous shape an evaluation can take, because it looks like
safety. Verdicts summarize; observations convince.

## Related

- <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::evaluation@agentic_development/reference/gates.md" />
- <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::evaluation@agentic_development/reference/datasets.md" />
- <ctxmod_link path="ctx:github.com/ctxmod/ai/context::shared::evaluation@agentic_development/reference/evaluation_first.md" />
