# Benchmark

Do the skills change what an agent writes? This harness answers that with evidence instead
of opinion: each task runs twice on a fresh Symfony project — once with the skills
installed, once without — and both diffs are scored against a binary checklist.

Be clear about what it is: **a repeatable demonstration, not a measurement instrument.**
Three reps per cell and an LLM judge can only show large, categorical differences — the
kind visible in the diffs anyway. That is enough for its purpose (anyone can re-run it and
watch the difference appear), and nothing subtler should be claimed from it. The findings,
and the history of how the tasks got their current shape, are in [RESULTS.md](RESULTS.md).

The tasks are the original complaint from the core team channel, made concrete. Each has a
Symfony component that covers it, and each is something an agent tends to hand-roll.

| Task | What it asks for | Skill under test |
|---|---|---|
| `00-control-value-object` | a `Money` type in plain PHP | **none — control** |
| `01-lock` | a cron command that cannot overlap | `command` |
| `02-json-endpoint` | a validated JSON endpoint | `controller` |
| `03-post-authorization` | only the author may edit a post | `voter` |

## Design rules

Each of these was learned by breaking it; the breakage is documented in
[RESULTS.md](RESULTS.md).

- **Every task maps onto a skill that exists here** — except the control, which must map
  onto none. A task about a component no skill mentions measures nothing, and a "control"
  that a skill covers devalues the whole run.
- **The control is plain PHP on purpose**: an amount, a currency, immutability, an
  exception on mismatch. No maker, no configuration, no routing, no template, no console.
  If the control shows a difference, the benchmark is measuring the setup — report that
  before any positive finding, and trust the run less.
- **Tasks must stand alone.** A task describes work on a fresh skeleton, so it may not
  claim anything already exists. An agent that spots a false premise may stop to ask — and
  in a non-interactive run that produces an empty diff, not an answer.
- **Checklists go to the judge verbatim.** `bin/judge` puts `CHECKLIST.md` into the
  prompt, so a checklist may not contain notes to the reader ("both variants should score
  the same here") — those become instructions to the judge.
- **A checklist item must be failable.** An item that checks for installing a package the
  skeleton already ships can never fail, and an item testing behaviour no skill teaches
  measures the model, not the skills.

## Running it

```bash
bin/run 02-json-endpoint without-skills
bin/run 02-json-endpoint with-skills
bin/judge 02-json-endpoint claude-opus-5
```

The second argument to `bin/judge` is the label `bin/run` wrote, so a judge call reads the
diffs it is meant to.

`bin/run` creates the project, installs the skills for the `with-skills` variant, hands the
task to the agent, and saves the diff under `results/`. It decides nothing.

`bin/judge` scores both diffs against `tasks/<task>/CHECKLIST.md` and prints JSON. The
checklist is **binary** — each item passed or it didn't, with the evidence quoted. A score
out of ten would just relabel a gut feeling, and the output has to survive being shown to
someone who disagrees.

An LLM does the scoring because the agent's output is not deterministic: two runs of the
same task differ in wording, file layout and naming while being equally right or equally
wrong. What stays comparable is whether `LockFactory` appears at all.

### Agents and models

`AGENT_KIND` selects how a run is set up; it decides where the skills are installed and how
the agent's own configuration is emptied.

```bash
bin/run 02-json-endpoint with-skills                       # claude, opus 5, the default
AGENT_CMD='claude -p --dangerously-skip-permissions --strict-mcp-config --model haiku' \
    bin/run 02-json-endpoint with-skills                   # same agent, weaker model

AGENT_KIND=opencode \
AGENT_CMD='opencode run --model ollama/kimi-k2.7-code' \
    bin/run 02-json-endpoint with-skills                   # different agent and model
```

Change one variable at a time. Swapping the agent and the model together produces a
difference nobody can attribute to either.

Under `AGENT_KIND=opencode` the skills are installed the way `docs/opencode.md` describes —
the folders under `.agents/skills` plus the index from `AGENTS.md` with its paths
adjusted — so a run also exercises the copy-only install path this repository documents.
Isolation is an empty `XDG_CONFIG_HOME`, the equivalent of the pristine
`CLAUDE_CONFIG_DIR` used for Claude Code.

Each configuration writes into `results/<task>/<label>/`, where the label is derived from
the model name unless `BENCHMARK_LABEL` overrides it — without a distinct label, a second
configuration would overwrite the first. A run that produces no agent output at all prints
a warning; empty diffs look like a result until someone checks, and silence is the failure
mode worth catching.

The model is pinned in the command and recorded in `evidence/`. The default is Opus 5, so
a plain `bin/run` continues the existing series rather than quietly starting a new one.
`JUDGE_CMD` overrides the judge the same way.

The default `AGENT_CMD` passes `--dangerously-skip-permissions`, because the agent has to
edit files without a human confirming each one. It runs against a throwaway project created
seconds earlier, and nothing else — read `bin/run` before you believe that.

## A clean environment

The agent runs under whatever Claude Code configuration it finds. Left alone that means
both variants inherit the operator's `~/.claude/CLAUDE.md`, the globally installed skills,
the plugins and any configured MCP servers — and then the baseline is not a baseline, it is
that machine. (On the machine this was written on, the global `CLAUDE.md` already carried a
Symfony instruction overlapping with the `cli-conventions` skill.)

So `bin/run` refuses to start without an isolated configuration. Create a credential once:

```bash
claude setup-token
export CLAUDE_CODE_OAUTH_TOKEN=<the token it prints>
```

Each run then gets a pristine `CLAUDE_CONFIG_DIR` — no user memory, no global skills, no
plugins, no MCP — and the session variables of any surrounding Claude Code session are
stripped from the child process. `ANTHROPIC_API_KEY` works in place of the token.

`BENCHMARK_ALLOW_INHERITED=1` runs anyway. Either way the mode lands in `environment.txt`
next to the diff, so a result can never claim to be clean when it was not.

Two further contaminations are removed rather than noted:

- The project is built in a temporary directory outside this repository. Built inside it,
  the skills would sit in an ancestor directory and be readable in both variants.
- `composer.lock` and `symfony.lock` are excluded from `diff.patch` — nearly all of the
  raw diff, and nothing a reviewer needs. What the agent installed stays visible in
  `packages.patch`.

## Reading the result

A checklist item that passes in both variants proves nothing about the skills; the model
knew it anyway. The interesting rows are the ones that pass only with the skills installed,
and the embarrassing ones are those that pass only without. A flat result is a result too:
it means the skill documents what the model already does — worth knowing when deciding
whether that skill earns its context lines.

Run each task more than once before concluding anything. A single run is an anecdote.

## Prerequisites

The Symfony CLI, PHP, Composer, `jq`, and an agent on `PATH`. Each run creates a full
Symfony project, so expect minutes and a network connection.

Everything under `results/` stays local — diffs, logs, verdicts and the timestamped
evidence files `bin/run` drops for each run. Conclusions belong in
[RESULTS.md](RESULTS.md), with the raw files of past runs preserved in git history.
Projects are built under `$TMPDIR/symfony-skills-benchmark`; override with
`BENCHMARK_WORKDIR`.

Agents tend to start the skeleton's `compose.yaml` for a database. `bin/run` shuts it down
afterwards, but check `docker ps` if a run is interrupted.
