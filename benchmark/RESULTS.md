# Benchmark results

Four runs, 2026-08-29 to 2026-08-31, all under the isolation `bin/run` enforces. This is
the consolidated record. The full per-run reports (`RESULTS.md` through `RESULTS-4.md`),
the raw verdict JSONs and the per-run evidence files live in the git history up to commit
`343b196`; nothing below says more than they do, only less.

## What stands

**One replicated effect.** Asked for a validated JSON endpoint (`02-json-endpoint`), the
agent with the skills installed used `#[MapRequestPayload]` in every rep; without them it
reached for `json_decode()` or manual `Serializer::deserialize()` and hand-assembled the
DTO. Three independent Opus 5 samples of three reps each: 3/3 vs 0/3, zero exceptions. A
fourth Opus sample found the single exception so far — one without-skills rep used the
attribute correctly on its own — and the effect held directionally on every other
configuration tried (Sonnet 5, Haiku 4.5, Kimi K2.7 Code and DeepSeek V4 Flash, the last
two under OpenCode via the copy-only install path in `docs/opencode.md`). A very strong,
model-independent effect; not an absolute one.

**Two honest nulls.** `03-post-authorization` (the `make-voter` skill): 12/12 in both arms,
twice independently — Opus writes idiomatic voters without being told. `01-lock` (the
`make-command` skill, with `symfony/lock` as the component under test): 12/12 in both arms once
the task premise was fixed. Neither is a failure of the benchmark; both mean the skill
documents what the strong models already do, which is an argument for keeping those skills
short — or asking whether they earn their context lines at all.

**A control that holds.** `00-control-value-object` — a plain-PHP value object no skill
covers — came back flat in both arms on Opus, Sonnet and Kimi, and showed only small,
directionless noise on Haiku and DeepSeek. The setup itself does not produce the
difference seen in `02`.

**A defect the benchmark found in the skills.** With the skills installed the agent kept
adding a `ValidationExceptionListener` that `#[MapRequestPayload]` makes redundant — the
attribute already answers 422 with the violations. `make-controller/SKILL.md` says so now.
Finding that is worth more than a third claimed success.

**A hint, not a finding.** On the fourth run's model ladder, two effects that are ceilinged
at Opus/Sonnet — the voter pattern and validator constraints on the DTO — flipped to
"with-skills reaches it, without-skills does not" at the Haiku 4.5 tier. Single 3-rep
samples, and at that tier the control itself starts losing resolution (10/12 in both arms),
so instrument noise and real effect are entangled. Recorded here in case someone wants the
"which models need this" question; this repository is not pursuing it.

## How the four runs got here

Each run's task set and checklist wording differ, so their numbers are records of their own
setups, not one growing sample. What each one changed:

1. **Run 1** built two of its three tasks around `symfony/lock` and Messenger, which no
   skill mentions — they measured nothing, and their flat results were briefly misread as a
   strong baseline. It also found two checklist items that could never fail because
   `symfony new --webapp` preinstalls the packages they checked for. The `02` effect showed
   up here first.
2. **Run 2** added a control that turned out not to be one: its premise claimed a console
   command already existed, so solving it meant creating one — which the `make-command` skill
   covers. The 12/12-vs-7/12 "control difference" was a skill doing its job under the wrong
   label. It also caught the without-skills agent stalling on the ambiguous premise, asking
   a question no one in a non-interactive run could answer.
3. **Run 3** introduced the plain-PHP control, fixed the `01-lock` premise, and delivered
   the shape reported above: the `02` effect a third time exactly, two flat nulls, a flat
   control. It also diagnosed the `ValidationExceptionListener` habit and fixed the
   `make-controller` skill.
4. **Run 4** ran the model/agent ladder (120 agent runs, 60 judge calls) and cost two
   infrastructure bugs on the way: a uutils-`env` argument-order difference that silently
   disabled isolation for one whole configuration, and a shared results directory that let
   configurations overwrite each other's verdicts. Both are fixed in `bin/run`. The ladder
   confirmed the `02` effect everywhere, produced the Haiku hint, and went further than the
   success criterion required — the with/without question was answered after run 3.

## Reading the numbers

Three reps per cell, an LLM as judge, a binary checklist: this instrument only sees large,
categorical differences — the kind visible to the naked eye in the diffs. It demonstrates;
it does not measure. Anything subtler than 3/3-vs-0/3 is inside its noise floor, and no
number here should be quoted with more precision than "consistently", "flat" or "noisy".
