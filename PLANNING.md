# Planning: Symfony Skills

A foundational set of Symfony best practices, written as skills for AI coding agents, so
that generated Symfony code looks like Symfony code — idiomatic, using the framework's own
components instead of hand-rolled substitutes.

## Problem

Coding agents working on Symfony projects build too much from scratch instead of reaching
for existing Symfony components and the documented best practices. Observed and discussed
in the Symfony core team channel.

## Goal

A prototype repository capturing the Symfony best practices as skills for coding agents —
easy to update, usable across agents, with no installation magic. The target is an official
Symfony repository; the prototype only goes to the core channel once it actually works, and
"works" has to mean something measurable (see [Success criterion](#success-criterion)).

## Prior art and attribution

[welcoMattic/symfony-skills](https://github.com/welcoMattic/symfony-skills) is the existing
skill set this work starts from. The repository is archived (last push 2026-03-26) and its
author explicitly allowed reuse. This repository starts fresh rather than continuing the
fork, and credits him as the original author in the README and here.

## Scope

|         |                                                                                                    |
|---------|----------------------------------------------------------------------------------------------------|
| **In**  | Best practices as skills, per-agent installation docs, an outlook document to start the discussion |
| **Out** | The skeleton's `AGENTS.md` (separate recipe PR), the distribution channel, per-component skills    |

## What a skill is here

Two kinds, and they are not interchangeable:

**Background knowledge** — conventions that apply whenever the agent touches Symfony code.
No steps, no templates, just what is true.

**Maker wrappers** — thin skills that point the agent at a MakerBundle command and show how
to invoke it non-interactively. They carry no PHP templates. Rebuilding `make:*` as an
embedded code template is exactly the sequencing that belongs in tooling, and the tooling
already exists.

### Which makers can actually be wrapped

Verified against `symfony/maker-bundle` (`main`): sixteen makers call `askQuestion` and
cannot be driven non-interactively — among them `MakeEntity`, `MakeForm`, `MakeUser`,
`MakeCrud` and `MakeAuthenticator`. `make:command`, `make:controller`, `make:migration`,
`make:voter` and `make:twig-extension` take their name as an argument and ask nothing.
(`MakeController` defines `interact()`, but it only computes — it never prompts.)

So the dividing line is a checkable fact, not a preference: **a maker gets a wrapper skill
if and only if it runs to completion without prompting.**

## Version one

Four background skills:

| Skill             | What it covers                                                        |
|-------------------|-----------------------------------------------------------------------|
| `cli-conventions` | `symfony console` preferred, `bin/console` as fallback                |
| `configuration`   | env vars vs. parameters vs. constants vs. secrets                     |
| `templates`       | Twig naming, fragments, AssetMapper                                   |
| `discover`        | Look it up in the project instead of recalling it (see below)         |

Plus maker wrappers for `command`, `controller`, `crud`, `entity`, `form`, `listener`,
`migration`, `test`, `twig-extension`, `user` and `voter`.

Entity and Crud were excluded while their makers needed a value that could not be passed in.
Both landed. `make:test` and `make:listener` were never blocked at all — they take everything
as arguments, and `make:listener` decides between a listener and a subscriber from the
class-name suffix. They were simply never checked, which is the same mistake that kept Form
and User out for a while.

## Blocking workstreams

Six pull requests against `symfony/maker-bundle` are merged,
[#1810](https://github.com/symfony/maker-bundle/pull/1810) through
[#1815](https://github.com/symfony/maker-bundle/pull/1815). They added `--field` and
`--relation` to `make:entity`, `--controller-class` to `make:crud` along with the crash that
made it unusable non-interactively, `--with-tests` to `make:controller`, and fixed the failing
tests on `1.x` on the way past.

Four makers still cannot run without a human, and three of them crash rather than fall back.
That framing is what carried the merged series: a command that accepts `--no-interaction` and
then dies is broken, not merely inconvenient.

| Maker | What is missing |
|---|---|
| `make:schedule` | three questions, no arguments at all; `$scheduleName` is read uninitialised |
| `make:auth` | all ten arguments are added inside `interact()`, so the definition is empty without it |
| `make:reset-password` | three asked values, plus three class guesses that live in `interact()` |
| `make:registration-form` | eight questions, eight uninitialised properties |

`make:docker-database` is deliberately left alone — infrastructure rather than a code pattern,
so no skill would wrap it either way.

## Structure

```
skills/
  <skill-name>/
    SKILL.md          # frontmatter + content
    ...               # optional resources, loaded only when needed
docs/
  <agent>.md          # installation docs per agent
  sources/            # interviews, research
AGENTS.md             # slim index for agents without a skill system
```

One folder per skill, frontmatter inside `SKILL.md` — not flat files with a central
metadata file. Metadata lives in exactly one place, and the folder leaves room for bundled
resources later.

Frontmatter is harmless for every target agent: Cursor and Copilot use frontmatter
themselves, and for Codex it is just a block at the top of the file.

### Frontmatter schema

| Field              | Purpose                                                      |
|--------------------|--------------------------------------------------------------|
| `name`             | identifier                                                   |
| `description`      | **the trigger condition** — see below                        |
| `version`          | so a change is attributable                                  |
| `updated`          | date of the last content change                              |
| `symfony-versions` | which Symfony versions this skill holds for — required       |

`description` deserves the attention. It is the only field an agent keeps in context for
every skill at once; everything else loads on demand. It therefore decides on its own
whether a skill is ever pulled in, and it is the most expensive text in the repository.

Write it as a trigger condition, not a summary of contents:

- ✅ "Use when creating a service class or wiring dependency injection."
- ❌ "Conventions for Symfony services."

## Installation: copy-only

No `install.sh`. Everything is documented instead.

- **Agents with a skill system (Claude Code and others):** copy the skills folder into the
  project. The agent reads every skill's frontmatter and pulls the full text on demand.
- **Agents without one (Codex and friends):** additionally copy the index block into
  `AGENTS.md` — a table of name, one-sentence description and file path, plus the
  instruction to read the file when relevant.

Deliberately not the archived repository's approach of concatenating every skill into a
single `AGENTS.md`: that loads everything on every turn. The slim index costs roughly 30
lines and reproduces on-demand loading in plain markdown.

Symfony version differences go inline in the skill text ("Symfony 6.4: `readonly class`
needs PHP 8.2, use individual `readonly` properties there") rather than into parallel
version folders. Without an install script nothing would resolve which of two files wins,
and the agent reads `composer.json` anyway.

## Relationship to the skeleton's `AGENTS.md`

Both must stand on their own. The `AGENTS.md` has to work for someone who never installs a
skill, and the skills have to work in a project whose `AGENTS.md` is missing or replaced.
Overlap is therefore a deliberate property of the design, not an oversight.

**In a conflict, the `AGENTS.md` wins.** Skills may deepen it; they may never contradict
it. A skill that disagrees with the skeleton file is a bug in the skill.

The rule exists because the two artefacts live in different repositories with different
review cycles, and they have already drifted once — over `symfony console` versus
`bin/console` — before a single line was written. The `AGENTS.md` ships inside a recipe and
changes rarely; letting the faster-moving side win would quietly turn the recipe into the
stale copy.

Detecting drift is a later stage, not version one: run the three benchmark tasks in three
configurations — `AGENTS.md` only, skills only, both. Diverging results are a contradiction,
found with machinery that already exists for the success criterion.

## Success criterion

Four tasks on a fresh `symfony new`, each run with and without the skills, diffs compared,
three repetitions per cell. Judged by an LLM — output is not deterministic — against a
**binary checklist**, not a scale. Everything mechanical is a script. The harness and the
reasoning behind it are in [`benchmark/`](benchmark/README.md).

Two rules were learned by breaking them, and both now hold:

- **Every task maps onto a skill that exists here**, except one that must map onto none. The
  first run's tasks were built around `symfony/lock` and Messenger, which no skill mentions;
  they measured nothing, and their flat results were misread as a strong baseline.
- **The control must be genuinely skill-free.** The second run's control moved and by the
  benchmark's own rule should have devalued everything else. It did not, because that task
  claimed a console command already existed — so solving it meant creating one, which the
  `command` skill covers. The control now asks for a value object in plain PHP.

### What three runs found

| | Result |
|---|---|
| `#[MapRequestPayload]` over `json_decode()` | **3/3 with skills, 0/3 without — three times, no exception** |
| Authorization in a voter | flat in both arms, twice |
| A cron command that cannot overlap | flat in both arms |
| The control | flat, on the run where it was finally a control |

One skill shows a clean, replicated effect on a task it targets directly. Two skills document
what the agent already does correctly. That is the honest shape of it, and the nulls are as
much a result as the difference — a benchmark that favoured the skills everywhere would be
measuring its own wishful thinking.

The runs also found something against the skills: with the skills installed the agent kept
adding a `ValidationExceptionListener` that `#[MapRequestPayload]` makes redundant. The
`controller` skill now says the attribute already answers 422. Finding that ourselves is
worth more than a third claimed success.

## Content and updates

The canonical source is <https://symfony.com/doc/current/best_practices.html>. Which areas
feed the skills is configurable.

The update path splits along the same line as everything else:

- **Script:** [`tools/update-skills/bin/fetch`](tools/update-skills/) downloads the
  configured sources, `bin/diff` prints the hunks that moved, grouped by affected skill.
- **Prompt:** `tools/update-skills/SKILL.md` judges whether a documentation change means a
  skill is now wrong.

That keeps the update mechanism subject to the repository's own writing rules instead of
exempting it.

There is **no update mechanism for copies already in someone's project**, and the plan does
not pretend otherwise. Someone who copied and adapted has no common ancestor to merge from.
Version and date in the frontmatter plus a readable changelog in the update PR is what we
offer. A real update path — override, enable/disable, a lockfile — is what Symfony Mate
brings, and that is where it belongs.

## Writing rules for skills

Taken from ["Un Skill n'est pas une lib"](https://devx.writizzy.blog/p/un-skill-nest-pas-une-lib)
(Frédéric Camblor).

Rule of thumb: *what requires judgement stays in the prompt; what is a sequence belongs in
the tooling.*

Anti-patterns that signal a skill is getting too complex:

- **Numbered steps** — a hidden sequencing requirement; belongs in a script.
- **Defensive phrasing** ("whatever you do, don't forget…") — papers over missing robustness.
- **Nested conditionals** — code in the wrong format.
- **Repeated re-prompting** to get the result right — non-determinism as a problem, not a
  feature.

Also: prefer a simple skill that fits exactly over one covering ten unnecessary scenarios
and bloating the context.

These rules apply to our own tooling too — which is why the maker wrappers carry no
templates and the update mechanism is a script.

## Positioning

Camblor's argument: skills are not libraries. They are context-bound, they age with the
models, and they resist meaningful versioning — there are no real regression tests, and one
person's feature is another's bug. His advice: fork and duplicate rather than converge.

We address this head-on:

- Camblor distinguishes four spheres, and the **"tool / product"** sphere — skills shipped
  with the software and maintained by its publisher — is the one he considers legitimate.
  That is exactly where an official Symfony repository sits.
- Our message was never "install this as a dependency" but: copy it in, adapt it when you
  need to.
- Skills therefore stay deliberately small and fork-friendly.

## Outlook (for the core discussion)

The prototype is ready to be argued with. It has one replicated effect, two honest nulls, a
control that holds, and a defect it found in itself. What it does not have is breadth: eleven
skills exist and three have been measured.

- **Widen the benchmark.** `configuration`, `templates` and `discover` are untested, and
  each makes a checkable claim.
- **Symfony Mate as the distribution channel.** Mate can override, enable and disable skills
  and keeps a lockfile, which is where a real update path belongs. Installing the `AGENTS.md`
  would fit there too.
- **Entity and Crud skills**, once the two maker options land.
- **Distribution** as a Composer package, a Claude Code plugin, or a Mate extension — open.

## Reference: `AGENTS.md` in the recipes PR

[symfony/recipes#1563](https://github.com/symfony/recipes/pull/1563) —
"[FrameworkBundle] Add AGENTS.md for AI coding agents", open. Not part of this repository,
but it sets the content scope and the quality bar.

The PR also adds a minimal `CLAUDE.md` containing `@AGENTS.md`, so there is a single source
of truth instead of duplicated content — the same principle we follow with the slim index.

Key points of the draft:

- Read `composer.json` and `symfony.lock` first; assume nothing about installed bundles.
- Ask when the task is unclear (persistence, interface, auth) rather than guessing; with no
  interactive channel, pick the smallest option and state the assumption.
- Install capabilities via Flex recipes; never hand-edit `config/bundles.php`.
- Conventions: attributes over YAML, autowiring, `readonly` DTOs, `symfony/validator`,
  `symfony/lock`, Messenger, Doctrine migrations, the secrets vault.
- When training data may be stale: read `vendor/` instead of relying on memory.

### Review feedback from javiereguiluz

Three suggestions worth adopting for the skill content:

1. **Split `Conventions`.** The section mixes "how to write code" with "how to work in the
   app". Proposal: a separate **"Everyday workflow"** section. For us that means workflow
   knowledge belongs in its own skills, not in the code-oriented ones.
2. **Generalize the conventions** and point at
   <https://symfony.com/doc/current/best_practices.html> instead of getting too specific.
   The key sentence: *"Before hand-writing infrastructure (locks, queues, caches, HTTP
   clients, mailers, schedulers…) or adding a third-party library, check whether a Symfony
   component covers it. It usually does."* — that is the answer to Alex's original problem
   and belongs prominently in our skills.
3. **A new "Discover, don't guess" section** — look it up in the project instead of
   recalling it: `bin/console about`, `debug:router`, `debug:container`,
   `debug:autowiring`, `debug:config`, `config:dump-reference`, `lint:container`,
   `lint:twig`, `lint:yaml`, plus reading `vendor/` and the docs version matching
   `composer.json`. This is the `discover` skill in version one.

### The skeleton's `AGENTS.md` could point here

The recipe file doesn't have to carry the skills — it only has to know they exist. A short
pointer ("for detailed Symfony conventions, see <skills repository>; copy the skills you
need into this project") gives every agent a discovery path without inflating what gets
loaded on every turn.

That gives the skills repository real distribution before any Composer package, plugin or
Mate extension exists — worth raising in the core discussion, since it makes the two efforts
complementary rather than competing.

### CLI conflict: resolved

The original draft used `bin/console` throughout, while the archived repository's
`cli-conventions` skill mandates `symfony console`. Javier's review comment resolves it:

> Run the app with `symfony serve -d`; run commands with `symfony console ...`
> (or `bin/console` if the Symfony CLI is not available).

We adopt that wording verbatim in `cli-conventions`, so the skeleton and the skills don't
contradict each other. Worth watching whether it lands in the PR as written.

## Open questions

- Does [maker-bundle#1810](https://github.com/symfony/maker-bundle/pull/1810) get accepted? The Entity skill depends on it.
- Does `make:crud` get a `--controller-class` option? The Crud skill depends on it.
- Does Javier's `symfony console` suggestion land in the recipes PR?
- Does this become an official Symfony repository, and via which distribution channel?

## Sources

- [symfony/recipes#1563](https://github.com/symfony/recipes/pull/1563) — `AGENTS.md` for the skeleton
- [welcoMattic/symfony-skills](https://github.com/welcoMattic/symfony-skills) — prior art (archived)
- [symfony/maker-bundle](https://github.com/symfony/maker-bundle) — which makers can be wrapped
- [Symfony Best Practices](https://symfony.com/doc/current/best_practices.html) — canonical content source
- ["Un Skill n'est pas une lib"](https://devx.writizzy.blog/p/un-skill-nest-pas-une-lib) — writing rules
