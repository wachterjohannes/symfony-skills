# Benchmark results — first run

3 reps per cell, 3 tasks × 2 variants × 3 reps = 18 agent runs + 9 judge calls.

- Agent: `claude -p --dangerously-skip-permissions --strict-mcp-config`
- Judge: `claude -p` (default)
- Isolation: `clean` in all 18 runs (verified per-run via `environment.txt`) — pristine
  `CLAUDE_CONFIG_DIR`, no inherited `CLAUDE.md`/skills/plugins/MCP servers.

## 01-lock

> A console command runs every minute from cron; overlapping runs double-send reminders.
> Make that impossible.

| Checklist item | with-skills | without-skills |
|---|---|---|
| Uses `symfony/lock` — `LockFactory` via DI | 2/3 | 2/3 |
| Does not hand-roll mutual exclusion | 3/3 | 3/3 |
| Adds via `composer require symfony/lock`, not by hand | 1/3 | 1/3 |
| Releases the lock, or relies on the component doing so | 2/3 | 3/3 |

**No consistent difference.** Both variants solve this task about equally well across 3
reps, and both hit the same items inconsistently:

- The "adds via `composer require`" item is evidence the judge could rarely confirm either
  way — it can only infer this from recipe artifacts (`.env` markers,
  `config/packages/lock.yaml`) appearing without a `config/bundles.php` change, and read
  that as pass or fail depending on the rep. This is a checklist-evidence problem, not a
  behavior difference.
- The one row that stands out: **rep 2, with-skills, failed both "uses LockFactory" and
  "releases the lock"** — the agent generated the bare `make:command` skeleton and never
  implemented locking at all, while the without-skills run in the same rep used
  `symfony/lock` correctly. This is the "skill made it worse" case the handoff doc asked
  to watch for. One occurrence in 3 reps; report it, don't generalize from it.
- Rep 3, without-skills used `LockableTrait` instead of an injected `LockFactory` — a
  legitimate `symfony/lock` usage the checklist's DI-specific wording marks as a fail. Also
  a checklist-wording issue, not a real gap.

## 02-json-endpoint

> `POST /api/subscribe` takes `email` + `plan`, validates both, 422 with details on invalid
> input, 201 on success.

| Checklist item | with-skills | without-skills |
|---|---|---|
| Reads the body via `#[MapRequestPayload]`, not `json_decode()` | **3/3** | **0/3** |
| Validates with `symfony/validator` constraints on a DTO | 3/3 | 3/3 |
| 422 comes from the framework, not hand-written error handling | **2/3** | **0/3** |
| Requires `symfony/serializer`/`symfony/validator` if missing | 0/3 | 0/3 |

**This is the finding.** Two rows differ, consistently, across all 3 reps:

- **`#[MapRequestPayload]` vs `json_decode()`**: with-skills used the attribute in every
  rep; without-skills used `json_decode()`/`$request->toArray()` and manually built the
  DTO in every rep. 3/3 vs 0/3, no exceptions.
- **Automatic 422 vs hand-written error handling**: with-skills relied on the framework's
  validation-failure response in 2 of 3 reps (rep 3 added its own
  `ValidationExceptionListener` — still correct behavior, just not the minimal path);
  without-skills hand-wrote a `foreach` over violations building the error array and
  response in all 3 reps.
- Both variants already know to put `#[Assert\Email]`/`#[Assert\Choice]` on the DTO — this
  row proves nothing about the skills.
- The "requires serializer/validator if missing" row fails in all 6 runs. Checked directly:
  a fresh `symfony new --webapp` already lists **both** `symfony/serializer` and
  `symfony/validator` in the root `composer.json`. Nothing is ever missing to require, so
  this item cannot pass on this skeleton — it isn't measuring the skills, it's unfailable
  by construction. Same class of problem as the `symfony/messenger` row in 03 below.

## 03-background-work

> A report page takes ~2 minutes to render; move the work off the request.

| Checklist item | with-skills | without-skills |
|---|---|---|
| Uses Messenger — message + `#[AsMessageHandler]` + `MessageBusInterface` | 3/3 | 3/3 |
| Does not build a queue by hand | 3/3 | 3/3 |
| Configures an async transport | 3/3 | 3/3 |
| Adds via `composer require symfony/messenger` | 0/3 | 0/3 |

**Identical in both variants, all 3 reps, all 4 rows.** This task does not discriminate at
all — matches the handoff doc's own warning that the baseline is strong. The 4th row fails
in all 6 runs for the same structural reason as 02's serializer/validator row: a fresh
`symfony new --webapp` already requires `symfony/doctrine-messenger`, which pulls in
`symfony/messenger` and ships `config/packages/messenger.yaml` before the agent starts.
Verified directly against a fresh skeleton. The item cannot pass here regardless of what
the agent does — drop it or rewrite it for a task where the component genuinely isn't
present yet (unlike `symfony/lock` in 01, which the skeleton does not ship).

## Summary

- **02-json-endpoint is the one task that discriminates**, and it does so consistently
  across all 3 reps: `#[MapRequestPayload]` (3/3 vs 0/3) and framework-driven 422 handling
  (2/3 vs 0/3) both favor with-skills with no counterexample.
- **01-lock and 03-background-work do not discriminate** at n=3. 01 has a strong baseline
  in both directions and checklist items whose evidence the judge can't reliably confirm
  either way. 03 is a flat ceiling — every run in both variants passes the 3 behavioral
  items and fails the 1 structurally-unfailable item.
- **One negative result**: 01-lock rep 2 with-skills produced no locking at all where the
  paired without-skills run did. Single occurrence, reported per the handoff doc's
  instruction to report rows that pass only without skills.
- **Two checklist items are miscalibrated**, not the skills or the agent: "requires
  symfony/serializer and symfony/validator if they are missing" (02) and "adds via
  composer require symfony/messenger" (03) both test for installing a package that
  `symfony new --webapp` already ships. Suggest dropping both or replacing them with an
  assertion that doesn't depend on the package being absent beforehand.
- Per the handoff doc's open question: with 3 reps, 2 of 3 tasks don't discriminate at all
  — before concluding the skills don't matter there, the checklists for 01 and 03 need
  sharper, more reliably-evidenced items (01) or a component the skeleton doesn't
  preinstall (03), then a re-run.

## Verification of this report

The three structural claims above were checked against `symfony/webapp-pack` on Packagist
rather than taken on trust:

| Package | In `webapp-pack` | Consequence |
|---|---|---|
| `symfony/serializer-pack` | yes | 02's "require if missing" item is unfailable — confirmed |
| `symfony/validator` | yes | same |
| `symfony/doctrine-messenger` | yes (pulls in `symfony/messenger`) | 03's require item is unfailable — confirmed |
| `symfony/lock` | **no** | 01's require item is legitimately failable — the task is sound |

`LockableTrait` was also checked: it holds a `LockFactory` and the `FlockStore` /
`SemaphoreStore` from `symfony/lock`, so rep 3's without-skills run was a real use of the
component that the checklist's DI-specific wording marked as a failure. A wording defect,
as reported.

The checklists have been corrected in the commit following this one. Those corrections take
effect on the **next** run — the numbers above were produced under the wording quoted here.

## What is not settled

The isolation claim rests on the `environment.txt` written beside each diff, and
`benchmark/results/` is deliberately not committed. Anyone re-running this should keep those
files, or the claim is unverifiable after the fact.

Task 03 has a ceiling problem no wording fix reaches: the component it tests for is
preinstalled, and both variants pass every behavioural item. Discriminating there needs a
component the skeleton does not ship. `symfony/scheduler`, `symfony/rate-limiter` and
`symfony/workflow` are all absent from `webapp-pack` and would each carry a task where the
agent has to recognise that a component exists at all. Choosing among them is a design
decision, not a fix, and is left open rather than settled here.
