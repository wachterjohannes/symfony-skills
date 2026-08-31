# Benchmark

Do the skills change what an agent writes? This answers that with evidence instead of
opinion: three tasks, each run twice on a fresh Symfony project — once with the skills
installed, once without — and both diffs scored against a checklist.

The tasks are the original complaint from the core team channel, made concrete. Each has a
Symfony component that covers it, and each is something an agent tends to hand-roll.

| Task | What it asks for | Skill under test |
|---|---|---|
| `00-control-value-object` | a `Money` type in plain PHP | **none — control** |
| `01-lock` | a cron command that cannot overlap | `command` |
| `02-json-endpoint` | a validated JSON endpoint | `controller` |
| `03-post-authorization` | only the author may edit a post | `voter` |

Every task maps onto a skill that actually exists here, except the control, which must map
onto none. Both halves of that were learned the hard way.

The first run's tasks were built around `symfony/lock` and Messenger, which **no skill here
mentions**. They could not have measured anything, and reading their flat results as "the
baseline is strong" was wrong — they were null tests.

The second run's control then moved, 12/12 against 7/12, which by the rule below should have
devalued the whole run. It did not, because that control was not one: its premise claimed a
console command already existed, so solving it meant *creating* one — which `command` covers
outright. A skill was doing its job on a task labelled as skill-free.

### The control

`00-control-value-object` is deliberately about nothing this repository teaches: an amount,
a currency, immutability, an exception on mismatch. No maker, no configuration, no routing,
no template, no console.

`cli-conventions` is the one skill an agent might still consult, since it will run Composer
and a test suite. How it invokes those does not appear in a diff, so it cannot move the
checklist.

If the control shows a difference, the benchmark is measuring the setup — sampling noise or
the mere cost of carrying eleven extra documents in context — and every other result in the
same run is worth less. Report that before any positive finding.

None of that may appear in `tasks/00-control-value-object/CHECKLIST.md`. `bin/judge` puts the
checklist into the judge's prompt verbatim, so a sentence there saying both variants should
score the same is an instruction to the judge, not a note to the reader. It was there for one
run; that run's flat control is weaker evidence than it looks, and the third run should be
repeated before its control is cited.

### Tasks must stand alone

A task describes work on a fresh skeleton, so it may not claim anything already exists.
`01-lock` did, and in one run the agent stopped to ask which of two readings applied — in a
non-interactive run, that produces an empty diff. It has been rewritten to ask for the
command outright, which breaks comparability with the first two runs on purpose.

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
the folders under `.agents/skills` plus the index from `AGENTS.md` with its paths adjusted —
so a run also exercises the copy-only install path this repository documents but has never
otherwise tried. Isolation is an empty `XDG_CONFIG_HOME`, the equivalent of the pristine
`CLAUDE_CONFIG_DIR` used for Claude Code.

Each configuration writes into `results/<task>/<label>/`, where the label is derived from the
model name unless `BENCHMARK_LABEL` overrides it. Without that, a second configuration
overwrites the first — the fourth run lost verdicts to exactly that.

A run that produces no agent output at all prints a warning. The fourth run lost an entire
configuration to an `env(1)` that stopped parsing `-u` flags at the first `NAME=value`
assignment: isolation quietly did nothing, and empty diffs look like a result until someone
checks. Silence is the failure mode worth catching.

The model is pinned in the command and recorded in `evidence/`. The first three runs did not
record theirs — they were Opus 5, which is the default here so that a plain `bin/run`
continues that series rather than quietly starting a new one.

`JUDGE_CMD` overrides the judge the same way.

The default `AGENT_CMD` passes `--dangerously-skip-permissions`, because the agent has to
edit files without a human confirming each one. It runs against a throwaway project created
seconds earlier, and nothing else — read `bin/run` before you believe that.

## A clean environment

The agent runs under whatever Claude Code configuration it finds. Left alone that means
both variants inherit the operator's `~/.claude/CLAUDE.md`, the globally installed skills,
the plugins and any configured MCP servers — and then the baseline is not a baseline, it is
that machine.

On the machine this was written on, the global `CLAUDE.md` already carried a Symfony
instruction (*"use `symfony composer <script>` commands"*), overlapping directly with the
`cli-conventions` skill. A result measured that way understates the skills and is fair game
for anyone who wants to dismiss it.

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
- `composer.lock` and `symfony.lock` are excluded from `diff.patch` — 89% of it in the
  first run, and nothing a reviewer needs. What the agent installed stays visible in
  `packages.patch`.

## Reading the result

A checklist item that passes in both variants proves nothing about the skills; the model
knew it anyway. The interesting rows are the ones that pass only with the skills installed,
and the embarrassing ones are those that pass only without.

Run each task more than once before concluding anything. A single run is an anecdote.

## Prerequisites

The Symfony CLI, PHP, Composer, `jq`, and an agent on `PATH`. Each run creates a full
Symfony project, so expect minutes and a network connection.

Evidence is committed, artifacts are not. Every run drops a timestamped file into
`results/<task>/evidence/` recording its isolation mode, agent and date; that directory
survives the clearing `bin/run` does on each invocation, so a repetition cannot erase the
record of the one before it. `diff.patch`, `agent.log` and the rest stay local. Projects are built under `$TMPDIR/symfony-skills-benchmark`;
override with `BENCHMARK_WORKDIR`.

Agents tend to start the skeleton's `compose.yaml` for a database. `bin/run` shuts it down
afterwards, but check `docker ps` if a run is interrupted.
