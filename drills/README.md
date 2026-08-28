# Research fluency drills

A personal training log, not a reproducibility artefact. Four drills, one file per day, no
registry, no generated index, no checker — see "Format" below for why that's on purpose.

## What this is for

AI can design and run an experiment faster than you can read what it did. That's fine for
execution; it's a problem when it quietly swallows judgment too — accepting a result because it
looks plausible, never forming your own opinion of whether it should be trusted. These drills keep
that judgment live without slowing the actual pipeline down: each one costs minutes.

## Relation to `experiments/`

Independent, not a dependency. `experiments/` records the experiment for anyone reading this repo
later — the ledger's `hypothesis` / `conclusion` fields are written to be read by someone else.
`drills/` is written to be honest with yourself right now, including admitting a guess was wrong.
Nothing here blocks `exp_ledger.py validate`, and nothing here should be pasted into a ledger field.

The only link is optional: a drill 3 entry may name a ledger ID so the two logs can be
cross-referenced, but nothing enforces it.

## The four drills, and when to run each

| # | Drill | Trigger |
|---|---|---|
| 1 | Load-bearing-line self-ID | An AI just wrote/handed you a script you didn't write. Guess the ~10-20 load-bearing lines *before* asking it to explain them. |
| 2 | Predict-before-verify | Before running a comparative/confirmatory experiment (skip on genuinely blind pilots). Predict the *shape* of the effect, not the binary sign — see the worked example in the skill file. |
| 3 | Code self-check | Right before you'd mark a result understood/closed. Paraphrase the load-bearing lines unaided, in 1-2 sentences. Can't write it → not closed yet. |
| 4 | Weekly hand-implementation | Once a week (or once per new module). Re-implement one bounded piece by hand, diff against the AI's version. |

Full rationale for each — why the order matters, worked examples, what each drill is *not* trying
to do — lives in `.claude/skills/research-fluency-drills/SKILL.md`. This README is the quick
reference; that file is the argument for why.

## Format

One file per day at `log/<day>.md`, started from `log/TEMPLATE.md`. Day-file split mirrors
`experiments/log/<day>.md` for the same reason: a single running file stops being navigable fast.

No JSON registry and no checker, unlike `experiments/registry/` — this log has one reader (you),
so nothing needs machine enforcement. Not every day has all four entries; drill 4 is weekly, and
some days have no new AI-written code to run 1-3 against. An empty day is not a failure. A week
with zero drill 4 entries is worth noticing.

```markdown
# Drills — YYYY-MM-DD

## Drill 1 — load-bearing-line self-ID
- Context: <what script/experiment>
- My guess: <file:lines, one clause why>
- AI confirmation: <hit / partial / miss, what I missed>

## Drill 2 — predict-before-verify
- Experiment: <what's being tested>
- Prediction (mechanism-level, not just sign): <...>
- Actual result: <...>
- Shape match?: <yes / no — if no, what does that imply about my model>

## Drill 3 — code self-check
- Ledger ID (if any): <...>
- My unaided paraphrase: <1-2 sentences>
- Closed?: <yes — sentence produced / no — flagged, not closing yet>

## Drill 4 — weekly hand-implementation (once/week)
- Module: <what>
- My implementation vs AI's: <diff summary, what was wrong on which side>
```
