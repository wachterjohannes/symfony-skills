# Benchmark results — second run (control + task swap)

3 reps per cell, 3 tasks × 2 variants × 3 reps = 18 agent runs + 9 judge calls.

- Agent: `claude -p --dangerously-skip-permissions --strict-mcp-config`
- Judge: `claude -p` (default)
- Isolation: `clean` on every run observed live during this batch (checked repeatedly while
  it was in progress). **Committed evidence is incomplete**: `bin/run` clears
  `results/<task>/<variant>/` on every invocation, and a per-rep archive step in the driver
  script copied `environment.txt` into the same directory it was about to be cleared from —
  so only the last rep's `environment.txt` per cell survived to be committed, not all
  three. The exception is `01-lock`/`without-skills`, which has both rep 2 (the stall,
  investigated separately below) and rep 3. A future run should archive per-rep files
  outside `results/<task>/<variant>/` from the start.

This run does not carry over run one's conclusions. `benchmark/RESULTS.md` stays as the
record of what was measured under the old task set and old checklist wording; the two are
not directly comparable except where noted below.

## The control (01-lock) first, as instructed

**The control shows a difference.** Per the handoff doc's own rule, that means every other
number in this run is worth less than it looks, and this has to be reported before any
positive result.

| Checklist item | with-skills | without-skills |
|---|---|---|
| Uses `symfony/lock` (DI or `LockableTrait`) | 3/3 | 2/3 |
| Does not hand-roll mutual exclusion | 3/3 | 1/3 |
| Adds via `composer require symfony/lock` | 3/3 | 2/3 |
| Releases the lock, or relies on the component doing so | 3/3 | 2/3 |

with-skills: 12/12. without-skills: 7/12.

**What actually happened, rep by rep:**

- **Rep 1**: both variants pass all 4 items. No difference.
- **Rep 2**: with-skills passes all 4. **without-skills produced an empty diff — the agent
  asked a clarifying question instead of writing any code, and stopped.** Reproduced on a
  second, independent attempt run specifically to check this wasn't a one-off (same
  result both times, `isolation: clean` both times, see
  `benchmark/results/01-lock/without-skills/environment-rep-2.txt`). The task's premise
  ("This application has a console command `app:send-reminders`...") doesn't hold on a
  fresh skeleton — the command doesn't exist yet and has to be built from scratch. In this
  rep, without skills, the agent noticed the mismatch, listed two options (wrong directory,
  or build it now), and ended its turn asking which one — which in a non-interactive `-p`
  run with no one to answer produces nothing. The with-skills run in the same rep hit the
  same missing-command situation and built it without asking.
- **Rep 3**: with-skills passes all 4. Without-skills used `LockFactory` correctly *and*
  additionally hand-rolled a compare-and-swap mutex via a `reminder_sent_at` database
  column (`DeadlineRepository::claimReminder()`) — belt-and-suspenders, not wrong, but a
  fail against "does not hand-roll mutual exclusion" since the component alone is supposed
  to be enough.

**Reading this honestly:** no skill in this repository documents `symfony/lock`, so this
task is a control by design — the skeleton should not affect *what* gets built, only
*whether* the agent commits to building it when the premise is ambiguous. Rep 2 is
consistent with that framing (a confidence/decisiveness effect, not a component-knowledge
effect — `discover`, the one always-on skill that isn't about a specific component, is the
only skill installed that plausibly touches this) but it is one occurrence, and one
occurrence is not a mechanism, it's an anecdote with a plausible story attached. The
honest conclusion is: **this control did not stay flat, the reason is not fully settled,
and the numbers for 02 and 03 below should be read with that in mind, not as independent
confirmations.**

## 02-json-endpoint (unchanged task, dropped checklist item, compare to run one)

Run one's third item ("requires serializer/validator if missing") was removed — confirmed
unfailable, see `RESULTS.md`. Three items remain.

| Checklist item | with-skills | without-skills | run one (for reference) |
|---|---|---|---|
| `#[MapRequestPayload]`, not `json_decode()` | 3/3 | 0/3 | 3/3 vs 0/3 |
| Validator constraints on a DTO | 3/3 | 3/3 | 3/3 vs 3/3 |
| Framework-driven 422, not hand-written | 1/3 | 0/3 | 2/3 vs 0/3 |

**Reproduced, direction unchanged, magnitude slightly weaker on one row.**
`#[MapRequestPayload]` vs `json_decode()` reproduced exactly: 3/3 vs 0/3, no exceptions,
same as run one. The 422-handling row still favors with-skills but by less this time (1/3
vs 2/3) — in reps 1 and 3, the with-skills agent added its own
`ValidationExceptionListener` even though `#[MapRequestPayload]` already fails with a 422
on its own; only rep 2 relied on the framework default. Both variants have always known to
put `#[Assert\Email]`/`#[Assert\Choice]` on the DTO — that row never discriminated in
either run.

Given the control finding above, treat "reproduced" here as "the pattern held up on a
second independent sample," not as proof the skills caused it — the same caveat applies
here as everywhere else in this run.

## 03-post-authorization (new task, replaces 03-background-work)

| Checklist item | with-skills | without-skills |
|---|---|---|
| Rule lives in a `Voter` (`supports()`/`voteOnAttribute()`) | 3/3 | 3/3 |
| Reached via `#[IsGranted]`/`denyAccessUnlessGranted()` | 3/3 | 3/3 |
| Supported attribute is a named constant | 3/3 | 3/3 |
| 403 comes from the security layer | 3/3 | 3/3 |

**Ceiling, identical in both variants, all 3 reps, all 4 rows.** Unlike the old
`03-background-work`, every row here is legitimately failable — the skeleton ships no
voter, and the `voter` skill exists precisely to teach this. The agent already builds
idiomatic voters without it. This is a real null result for this task, not a checklist
defect: the baseline for "put an authorization rule in a Voter reached via `#[IsGranted]`"
is already at ceiling for this agent, at least at n=3.

## Summary

- **Lead finding: the control was not flat.** `01-lock` — a task no skill here covers —
  showed with-skills at 12/12 and without-skills at 7/12, driven mostly by one
  without-skills run that stalled asking a clarifying question instead of building against
  an ambiguous premise (reproduced once independently), plus one run that hand-rolled a
  redundant mutex alongside correct `symfony/lock` usage. Per the handoff doc's own rule,
  this means the whole run's positive results below carry less weight than they would if
  the control had been flat.
- **02-json-endpoint reproduced its headline finding** (`#[MapRequestPayload]`: 3/3 vs
  0/3, exact match to run one) and its secondary finding held in weaker form (422 handling:
  1/3 vs 0/3, down from 2/3 vs 0/3). Given the control result, "reproduced" should be read
  as "consistent across two samples," not as settled.
- **03-post-authorization is a clean ceiling** — both variants already write idiomatic
  voters. A real null result for a task where the checklist itself is sound (unlike the old
  03).
- **What this run does not settle:** whether `01-lock`'s control difference is a genuine
  decisiveness effect from the `discover` skill, sampling noise from a small n, or an
  artifact of how a `-p` non-interactive run handles an agent that asks a question. A third
  independent run of `01-lock` without-skills specifically — enough reps to see whether the
  stall recurs at some stable rate or was a fluke — would settle more than widening to a
  fourth task right now.
