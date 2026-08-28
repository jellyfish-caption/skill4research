---
name: research-fluency-drills
description: Run the four self-training drills — load-bearing-line self-ID, predict-before-verify, code self-check, and the weekly hand-implementation drill — whenever an AI-designed experiment or AI-generated code is about to be read, run, or accepted. Use before opening an AI-written verify/ablation script, before marking a ledger entry's conclusion as understood (not just recorded), and once a week for the hand-implementation drill. Distinct from the `experiment-ledger` skill: the ledger records the experiment for reproducibility; this skill trains the human running it.
---

# Research fluency drills

## Why this exists

AI can design and run an experiment faster than a first-time researcher can read what it did.
That gap is not itself a problem — offloading execution has always been normal in research. The
problem is when the gap silently swallows judgment too: skimming a result, trusting it because it
*looks* plausible, and moving to the next decision without ever forming an opinion of your own
about whether it should be trusted. That failure mode doesn't show up in the paper. It shows up
later — in a quals room, in front of a new advisor, or the first time you're handed someone else's
undocumented baseline and have to extend it without an AI already knowing what's load-bearing.

These four drills exist to keep that muscle live while the AI-assisted pipeline runs at full
speed. None of them require reading full files or slowing down the pipeline meaningfully — each
is scoped to cost minutes, not hours, because a drill that gets skipped under deadline pressure is
worthless.

**This is not the `experiment-ledger` skill.** The ledger's `hypothesis` / `conclusion` fields are a
record for other readers — they can be filled in correctly by someone who understood the result
well enough to write a sentence about it, which is a lower bar than actually having formed a
falsifiable expectation beforehand or being able to explain the mechanism unaided. The drills below
are that higher bar, for yourself only. Nothing here is machine-checked, and nothing here should be
copy-pasted into a ledger field — a ledger `hypothesis` is written to be read by someone else later;
a drill entry is written to be honest with yourself now, including admitting when a guess was wrong.

## The four drills

### 1. Load-bearing-line self-ID — before reading an AI explanation of new code

**Trigger**: an AI has just written or handed you an experiment/verification script you didn't
write yourself.

**Action**: before asking the AI to explain it or point anything out, guess which ~10–20 lines are
load-bearing — the ones that actually compute the paper's core number, or decide what's being
compared to what. Write the guess down (file + line range + one clause on why you picked them).
Then ask the AI to confirm or correct. Record hit / partial / miss.

**Why this order matters**: if the AI points at the important lines first, you get faster at
reading *pre-labeled* important code, which is not the skill you need later — the skill you need is
recognizing what's likely to matter in code nobody has labeled for you yet (an open-source baseline
with no docs is the real test). The guess is the whole exercise; accuracy is not the point at first.

### 2. Predict-before-verify — before running a comparative or confirmatory experiment

**Trigger**: an AI is about to run something that has a real answer you don't already know —
excludes genuinely blind exploratory pilots where you have no mechanism hypothesis at all; don't
force a prediction where none exists.

**Action**: write a prediction that is **not** the top-line binary outcome ("A beats B" is close to
a coin flip and low-information even when you're right). Write a mechanism-level prediction instead:
under which subset, condition, or slice do you expect the effect to concentrate, and why. Example —
testing whether adding retrieval helps: not "I predict it helps," but "I predict the gain
concentrates on queries needing stale/rare facts, and is flat-to-negative on queries answerable from
parametric knowledge alone, because retrieval is compensating for a knowledge gap, not adding
reasoning capacity." After the result, check the *shape*, not just the sign. A prediction that got
the top-line direction right but the shape wrong means your model of *why* it works is wrong even
though you "won" — that mismatch is the highest-value five minutes in the whole entry.

### 3. Code self-check — before closing out any experiment as understood

**Trigger**: right before you'd otherwise mark a ledger entry `complete` or move on to the next
decision.

**Action**: write one or two sentences, unaided, paraphrasing what the load-bearing lines from
drill 1 actually do. No AI drafting this sentence — you can use AI *afterward* to check it, not to
write it. If you can't produce the sentence, the experiment isn't understood yet, and the entry
doesn't close — regardless of whether the ledger's required fields are technically filled in.

**This is the drill most likely to get skipped under deadline pressure — treat that as the signal,
not the excuse.** If you notice yourself reaching for "the result looked fine, next" instead of
writing the sentence, that is exactly the moment the black-boxing described in "why this exists"
above is happening in real time.

### 4. Weekly hand-implementation drill

**Trigger**: once a week, or once per meaningfully new module, whichever comes first.

**Action**: pick one bounded function or module from that week's AI-generated pipeline code — a
loss function, a preprocessing step, a metric computation. Re-implement it by hand, without AI
assistance, from your own understanding of what it should do. Then diff against the AI's version.
Note every discrepancy and figure out which side was right.

**This is a training method, not an operational requirement.** The goal is not "you must be able to
hand-write this in production" — it's that writing something once builds the pattern recognition
that makes *reading* similar code fast later, the same way practicing writing in a language speeds
up reading it. Keep the scope small on purpose; a drill that eats an afternoon will stop happening
the first week things get busy.

## The log

Kept at `drills/log/<day>.md`, one file per day with an entry per drill run — mirrors the
`experiments/log/<day>.md` convention in this repo for the same reason: a day file stays navigable,
a single running file doesn't. Unlike the ledger, there is no registry, no generated index, and no
checker — this is a personal record, not a reproducibility artefact, and adding that machinery here
would be solving a problem this doesn't have.

Template for a day file:

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

Not every day has all four entries — drill 4 is weekly, and some days have no new AI-written code
to run drill 1/2/3 against. An empty day file is not a failure; a week with zero drill 4 entries is
worth noticing.

## What this does not do

It does not slow down the actual research pipeline — every drill is scoped to minutes. It does not
replace the ledger's `hypothesis`/`conclusion` fields, which serve a different reader. It does not
guarantee catching every bug an AI-designed experiment might have — that's a separate, deeper skill
(auditing for data leakage, metric miscomputation, unfair baseline configs) that this repo's
practice has concluded is best built *on top of* drill 1 fluency rather than as a parallel track:
once load-bearing lines are second nature to spot, tracing them for leakage is the same skill
applied one step further, not a new one to schedule separately.
