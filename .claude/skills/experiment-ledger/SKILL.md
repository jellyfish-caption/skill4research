---
name: experiment-ledger
description: Keep a machine-checked experiment ledger complete and current in a research repo. Use whenever an experiment is planned, launched, finishes, fails, is superseded, or is being looked up — "what has been run", "why did we run that", "where are the results" — or when writing a result into a narrative doc. Enforces one record per experiment covering source, attachment point, question, pre-registered hypothesis, launch command, artefact paths, visualization and conclusion, with dates on everything.
---

# The experiment ledger

## Why this exists

A narrative experiment log is good at telling a story and bad at answering *what has ever been
run, and is any of it stale?* — this pattern grew out of a repo where several completed runs had
quietly stopped appearing in the narrative log at all. Prose degrades silently; a record with
required fields does not.

Duty is split three ways and the split is the whole design:

| | holds | source of truth |
|---|---|---|
| `experiments/registry/<created>-<ID>.json` | the facts needed to **find** and **re-run** the work | yes |
| a narrative-doc section (one per research question/thread) | the **argument** — why the result means what it means | yes, for prose |
| `experiments/INDEX.md` | the rendered table | **no. Generated. Never hand-edit.** |
| `experiments/log/<day>.md` | hand-written narrative **plus** a projected block per record | prose yes; `<!-- ledger:ID -->` blocks no |
| `experiments/EXPERIMENT_LOG.md` | the index over the day files | **no. Generated.** |

Records are named `<created>-<ID>.json` so a plain `ls experiments/registry` reads as a timeline.
The date prefix is fixed width, so splitting it off the front is unambiguous even when IDs
themselves carry dots and hyphens.

**The narrative log is not retired, and you no longer type its structured half.** It is one file
per day under `experiments/log/`, because a single running file stops being navigable once a
project has been going a few weeks and a busy day has a dozen entries. A ledger tool projects each
record into the day file for its `created` date, inside a `<!-- ledger:ID -->` block, and
regenerates `EXPERIMENT_LOG.md` as the index over the day files. Re-running rewrites a block in
place rather than appending a second copy, and text outside the markers is never read or modified
— so hand-written prose survives on both sides of a block.

**Write prose in the day file.** The launch command, job ids and artefact paths come from the
record; the argument does not, and never will.

The rules live in a small library module and are executed by a CLI wrapper and a test suite, not
in this file — so they hold for any agent or human touching the repo, not only for a session that
happened to load this skill.

## Commands

```bash
python scripts/exp_ledger.py status                       # what is running, stale, and open
python scripts/exp_ledger.py new EXP-042 "One-line title"  # scaffold; it will FAIL validation
python scripts/exp_ledger.py render                       # regenerate experiments/INDEX.md
python scripts/exp_ledger.py log                           # project records into experiments/log/<day>.md
python scripts/exp_ledger.py validate                      # contract + index freshness; exit 1 on error
python -m unittest tests.test_ledger                       # the contract's own audit
```

`render` then `log` after any record change; `validate` will tell you if you forgot.

Stdlib only is a deliberate constraint, not an accident — it's what lets the ledger run anywhere
the experiments themselves run, including a remote machine with no package installs available.

## The three moments you must write

### 1. Before launching — open the record

`new` writes a scaffold whose required fields are empty **on purpose**. A template that validates
while still unanswered trains everyone to leave it unanswered.

Fill in, and none of these is optional:

- **`question`** — what is being asked, in plain language a reader who did not watch the work can
  follow. Not "run the script"; that is the launch command, not the question.
- **`decision`** — what this gates. If you cannot name a decision that changes with the answer,
  reconsider whether to spend the compute.
- **`attaches`** — where it plugs into the existing chain. Draw it explicitly: which stage, which
  upstream artefact, and **why the existing flow could not already do it**.
- **`hypothesis`** — **which outcome means what, written before the numbers exist.** The checker
  refuses to let a record leave `planned` without it. A rule fixed only after seeing the data it
  was meant to gate is the same failure in a different costume, no matter how many times a project
  has already learned that the hard way.
- **`source`** — repo-relative paths. Checked to exist. Prefix `remote:` for anything that lives
  only on a remote machine.
- **`narrative`** — which narrative-doc section carries the argument.

Then `render`, `log` and `validate` before you submit anything.

### 2. At launch — record how

Set `status: running`, fill `launch` with the **exact** command (the checker requires it at this
status), add the job IDs to `jobs`, and add a `history` line. Then `render` and `log`.

A job that fails and is resubmitted keeps **both** ids in `jobs` and gets a `history` line saying
what broke — dropping the dead id would erase the only record that the failure ever happened. The
shape to copy: a run that dies in seconds on a trivial argv/config collision, resubmitted with the
one-line fix, both ids kept in the same record.

### 3. When it finishes — the part that actually gets skipped

Set the terminal status and fill:

- **`finished`** — required by the checker for every terminal status.
- **`artifacts`** — where the results live. Required for `complete`. Remote paths take the
  `remote:` prefix; a small reviewed copy under `artifacts/review/` may be listed alongside.
- **`visualization`** — *how the result was made legible*, not a promise to make a figure later.
  A named table, an ASCII histogram, a per-fold grid all count. The bar is that a reader can see
  the result without re-deriving it. The shape to copy: a plain histogram that revealed a bimodal
  split invisible in any pair of order statistics — printing it is what caught a claim that had
  run ahead of its evidence.
- **`conclusion`** — the **scoped** conclusion. Carry the caveat that cuts against it in the same
  field. The shape to copy: "passed on the letter of the rule … must NOT be read as a clean pass,
  because …" — not a specific result, a template for how a scoped conclusion reads.

Then write the argument into the narrative-doc section named in `narrative`, then `render`, `log`
and `validate`.

## What the checker enforces, and what it cannot

**Enforced** (`validate` exits 1): required prose present, long enough, and free of `TBD` / `TODO`
/ `n/a` / placeholder filler; `hypothesis` before leaving `planned`; `conclusion` + `artifacts` +
`finished` + `visualization` before `complete`; `launch` while `running`; `source` paths exist;
dates ordered and not in the future; `superseded` names its successor and cross-references
resolve; the rendered index and the log index match what they are generated from; record
filenames carry the right date prefix.

**Warned, not enforced**: a `running` record untouched for more than 7 days — a long job is
legitimate, a dead job looks exactly like one, and only the ledger tells them apart cheaply; and a
missing or stale block in a day file, which is untidy rather than wrong because those files are
mostly hand-written.

**Not checkable, and therefore yours**: whether the hypothesis was really registered before the
numbers were seen, whether the conclusion is scoped to what was measured, and whether the caveat
that cuts against the result is stated. The checker can tell that a conclusion field is non-empty.
It cannot tell that it is honest.

## Rules that matter more than the schema

- **A failed run is a record, not a deletion.** A run that loses most of its evaluations to a
  crash, and a run whose first submission dies on a bad assertion, both belong in the ledger with
  `status: failed` or a `history` line — the failure carries the information.
- **A retraction is a record.** A record can exist solely to document a hypothesis that died.
  Write the death in the `conclusion`; do not quietly weaken the claim into something that still
  sounds interesting.
- **Superseding, never deleting.** Set `superseded_by` and keep the old record. The checker
  verifies the successor exists.
- **Never promote an unrun idea, a proxy-only result, a missing value, or a tool failure into a
  measured conclusion.** Worth writing into the repo's own standing-rules doc if it has one; the
  ledger is where this is easiest to break by accident.
- **`history` kinds are fixed**: `Observed`, `Interpretation`, `Decision`, `Open question`, `Next`.
  `Open question` entries surface in `status`, which is how they stop being forgotten.

## Long-term maintenance

Run `python scripts/exp_ledger.py status` at the start of any experiment session — before reading
the log, because it is orders of magnitude shorter and it tells you what is live.

`validate` belongs in whatever pre-commit or CI check the repo grows, since it fails on a broken
record and on either generated file drifting. A test that runs the contract over the real registry
catches drift the day it happens, not the day someone notices the index looks wrong.

When a record's shape genuinely does not fit — a new required field, a new status — change the
library module and add the test that proves the new rule fires. Do not work around the schema by
stuffing prose into a field that means something else.
