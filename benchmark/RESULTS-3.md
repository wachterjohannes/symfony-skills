# Benchmark results — third run (real control, 01-lock reclassified)

4 tasks × 3 reps × 2 variants = 24 agent runs, 12 judge calls.

- Agent: `claude -p --dangerously-skip-permissions --strict-mcp-config`
- Judge: `claude -p` (default)
- Isolation: `clean` on all 24 runs, verified from the committed
  `benchmark/results/<task>/evidence/` files (one per run, never overwritten — the archiving
  bug from run two is fixed in `bin/run` itself this time).

Does not carry over earlier conclusions. `RESULTS.md` and `RESULTS-2.md` stand as their own
records under their own task/checklist wording.

## 00-control-value-object — the control, first

**Flat.** 4/4 on both variants, all 3 reps, no exceptions:

| Checklist item | with-skills | without-skills |
|---|---|---|
| Amount is an integer in minor units | 3/3 | 3/3 |
| Immutable, no setters | 3/3 | 3/3 |
| Currency mismatch throws | 3/3 | 3/3 |
| Exception covered by a test | 3/3 | 3/3 |

No skill here covers plain-PHP value objects, and none of the four items is
structurally unfailable (each is a real, checkable design choice — checked: `final`
and named class constants, the only plain-PHP vocabulary the skills use anywhere, do not
appear in this checklist by design). This control did what a control is supposed to do.
**The rest of this run's numbers can be read at face value.**

## 02-json-endpoint — third independent sample

| Checklist item | with-skills | without-skills | run 1 | run 2 |
|---|---|---|---|---|
| `#[MapRequestPayload]`, not `json_decode()` | **3/3** | **0/3** | 3/3 vs 0/3 | 3/3 vs 0/3 |
| Validator constraints on a DTO | 3/3 | 3/3 | 3/3 vs 3/3 | 3/3 vs 3/3 |
| Framework-driven 422, not hand-written | 0/3 | 0/3 | 2/3 vs 0/3 | 1/3 vs 0/3 |

**The headline claim reproduced a third time, exactly: 3/3 vs 0/3, zero exceptions across
three independent samples of three reps each (nine agent runs total on each side).** This
is the strongest single claim in this project. With-skills consistently wrote
`#[MapRequestPayload] SubscribeRequest $request`; without-skills consistently used
`json_decode()` or manual `Serializer::deserialize()` and hand-assembled the DTO — every
single time, in every run, over three independent samples.

The 422-handling row has now declined across all three runs (2/3 → 1/3 → 0/3
with-skills, flat at 0/3 without-skills throughout) and stopped discriminating this time.
Looking at what actually happened: the with-skills agent still never relies on the
framework's bare default — it either adds its own `ValidationExceptionListener` (2 of 3
reps here) or the diff simply doesn't show enough to confirm the 422 path was exercised at
all (1 rep). This looks like a stable habit (with-skills agents build an explicit listener
as a matter of course) rather than a regression, but three declining numbers in a row is
also the kind of pattern that deserves a sharper checklist item rather than continued
trust — e.g. asserting on whether a 422 test exists and passes, rather than inferring
implementation style from the diff alone.

Validator-on-DTO has never discriminated in three runs; both variants always knew this.

## 01-lock — first observation under the new premise, not compared to earlier runs

The task now asks outright for a command that doesn't exist yet, and the checklist is
unchanged from run 2's wording. This tests the `command` skill, not a component-only
control.

| Checklist item | with-skills | without-skills |
|---|---|---|
| Uses `symfony/lock` (DI or `LockableTrait`) | 3/3 | 3/3 |
| Does not hand-roll mutual exclusion | 3/3 | 3/3 |
| Adds via `composer require symfony/lock` | 3/3 | 3/3 |
| Releases the lock, or relies on the component doing so | 3/3 | 3/3 |

**Flat. 12/12 both variants, all 3 reps.** Unlike run 2, no rep stalled on the premise this
time — every run, in both variants, built the command outright (constructor-injected
`LockFactory` or `LockableTrait`, correct release, correct recipe install) without asking
anything. Given the control above is also flat and this run's isolation is fully verified,
this reads as a genuine null, not noise: at n=3, the agent already knows how to build a
locked cron command and reach for `symfony/lock`, with or without the skill. First
observation under the new premise — one run is not three independent samples the way 02's
finding is, so "settled" is premature, but there is no signal here to chase yet either.

## 03-post-authorization — second flat result

| Checklist item | with-skills | without-skills |
|---|---|---|
| Rule lives in a `Voter` | 3/3 | 3/3 |
| Reached via `#[IsGranted]`/`denyAccessUnlessGranted()` | 3/3 | 3/3 |
| Supported attribute is a named constant | 3/3 | 3/3 |
| 403 comes from the security layer | 3/3 | 3/3 |

Identical to run 2: 12/12 both variants, all 3 reps. **Two independent flat results now.**
The checklist is sound (skeleton ships no voter, every item is legitimately failable), so
this is a settled null for the `voter` skill: this agent already writes idiomatic
authorization voters without being told, and a third run of this exact task is unlikely to
change that conclusion.

## Summary

**This run gives the project what it needs for the Symfony core channel: one clean,
repeated, isolated difference, plus honest null results for two skills, on a control that
held.**

- **00 (control): flat**, as it should be — the setup itself is not producing the
  difference seen elsewhere.
- **02 (`#[MapRequestPayload]`): reproduced a third time, exactly**, 3/3 vs 0/3 with zero
  exceptions across nine runs per side. This is the one finding worth presenting on its own
  merits, not hedged by a moving control this time.
- **01 (`command`/`symfony/lock`) and 03 (`voter`): both flat.** 03 is now a settled null
  across two independent runs; 01 is a first flat observation under its corrected premise.
  Neither is a failure of the skills — they document what the agent already does correctly,
  which is exactly the kind of honest negative result a benchmark should be able to produce
  without spinning it.
- The 422-handling row in 02 is the one loose thread: three runs, a declining but never
  reversing pattern, worth a sharper checklist item (assert on a passing 422 test rather
  than inferring style from the diff) before drawing anything further from it.

Overstating this would mean claiming three skills work; the honest claim is one skill shows
a clean, three-times-replicated effect on a task it targets directly, and the benchmark
correctly produced two flat nulls elsewhere rather than manufacturing false positives.
