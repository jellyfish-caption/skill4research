# skill4research

Constraints and structural planning for AI research before getting overwhelmed.

Two independent Claude Code skills for working with an AI-assisted research pipeline: one keeps
the experiment record straight, the other keeps the researcher's own judgment and code fluency
from quietly eroding while the AI does the execution.

## What's here

| | what it is |
|---|---|
| `.claude/skills/research-fluency-drills/SKILL.md` | Four drills — load-bearing-line self-ID, predict-before-verify, code self-check, weekly hand-implementation — for staying able to read, predict, and audit AI-generated experiment code instead of just consuming its conclusions. |
| `drills/` | The log format and template the drills write to. |
| `.claude/skills/experiment-ledger/SKILL.md` | A machine-checked experiment ledger: one record per experiment (question, hypothesis, launch command, artefacts, conclusion), a generated index, and a day-file narrative log — so "what has been run and is any of it stale" has an answer that doesn't degrade like prose does. |

## How to use

Drop `.claude/skills/` into any repo Claude Code works in and both skills load automatically —
Claude reaches for `experiment-ledger` around planning/launching/closing an experiment, and for
`research-fluency-drills` around reading or accepting AI-generated code and results. Neither
skill requires the other; `experiment-ledger` assumes an `experiments/registry` + `scripts/`
layout described inside it, `research-fluency-drills` only assumes a `drills/log/<day>.md` you
start from `drills/log/TEMPLATE.md`.

They also work read-only, without Claude in the loop: `research-fluency-drills`'s four triggers
are just things to do by hand at the right moment, and `experiment-ledger`'s schema is documented
well enough to fill in a JSON record without the CLI tool it references.

## Motivation

<!-- fill in: what problem this was built to solve, and how it came about -->
