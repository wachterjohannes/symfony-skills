# Benchmark

Do the skills change what an agent writes? This answers that with evidence instead of
opinion: three tasks, each run twice on a fresh Symfony project — once with the skills
installed, once without — and both diffs scored against a checklist.

The tasks are the original complaint from the core team channel, made concrete. Each has a
Symfony component that covers it, and each is something an agent tends to hand-roll.

| Task | What it asks for | What it should reach for |
|---|---|---|
| `01-lock` | stop a cron command from overlapping | `symfony/lock` |
| `02-json-endpoint` | a validated JSON endpoint | `#[MapRequestPayload]` + Validator |
| `03-background-work` | move slow work off the request | Messenger |

## Running it

```bash
bin/run 01-lock without-skills
bin/run 01-lock with-skills
bin/judge 01-lock
```

`bin/run` creates the project, installs the skills for the `with-skills` variant, hands the
task to the agent, and saves the diff under `results/`. It decides nothing.

`bin/judge` scores both diffs against `tasks/<task>/CHECKLIST.md` and prints JSON. The
checklist is **binary** — each item passed or it didn't, with the evidence quoted. A score
out of ten would just relabel a gut feeling, and the output has to survive being shown to
someone who disagrees.

An LLM does the scoring because the agent's output is not deterministic: two runs of the
same task differ in wording, file layout and naming while being equally right or equally
wrong. What stays comparable is whether `LockFactory` appears at all.

Both commands take an override, so this is not tied to one agent or one judge:

```bash
AGENT_CMD='codex exec' bin/run 01-lock with-skills
JUDGE_CMD='claude -p --model opus' bin/judge 01-lock
```

The default `AGENT_CMD` passes `--dangerously-skip-permissions`, because the agent has to
edit files without a human confirming each one. It runs against a throwaway project created
seconds earlier, and nothing else — read `bin/run` before you believe that.

## What both variants inherit

The agent runs under the user's own Claude Code configuration, so both variants inherit
whatever is in `~/.claude/CLAUDE.md`, the globally installed skills, and any configured MCP
servers. That is a constant across the two arms and does not invalidate the comparison, but
it does bound what the numbers mean.

It matters here more than usual: this machine's global `CLAUDE.md` already contains a
Symfony instruction (*"use `symfony composer <script>` commands"*), which overlaps with the
`cli-conventions` skill. The baseline is therefore stronger than a naive agent's, and any
measured effect of the skills is a **lower bound**.

Two contaminations are removed rather than merely noted:

- The project is built in a temporary directory outside this repository. Built inside it,
  the skills would sit in an ancestor directory and be readable in both variants.
- `composer.lock` and `symfony.lock` are excluded from `diff.patch` — they were 89% of it
  in the first run and say nothing a reviewer needs. What the agent installed stays visible
  in `packages.patch`.

For a genuinely clean baseline, run the agent under a separate `CLAUDE_CONFIG_DIR` with its
own login. That is not wired in here, because the second profile has to be authenticated by
hand.

## Reading the result

A checklist item that passes in both variants proves nothing about the skills; the model
knew it anyway. The interesting rows are the ones that pass only with the skills installed,
and the embarrassing ones are those that pass only without.

Run each task more than once before concluding anything. A single run is an anecdote.

## Prerequisites

The Symfony CLI, PHP, Composer, `jq`, and an agent on `PATH`. Each run creates a full
Symfony project, so expect minutes and a network connection.

`results/` is not committed. Projects are built under `$TMPDIR/symfony-skills-benchmark`;
override with `BENCHMARK_WORKDIR`.

Agents tend to start the skeleton's `compose.yaml` for a database. `bin/run` shuts it down
afterwards, but check `docker ps` if a run is interrupted.
