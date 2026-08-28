# Benchmark

Do the skills change what an agent writes? This answers that with evidence instead of
opinion: three tasks, each run twice on a fresh Symfony project — once with the skills
installed, once without — and both diffs scored against a checklist.

The tasks are the original complaint from the core team channel, made concrete. Each has a
Symfony component that covers it, and each is something an agent tends to hand-roll.

| Task | What it asks for | Skill under test |
|---|---|---|
| `01-lock` | stop a cron command from overlapping | **none — control** |
| `02-json-endpoint` | a validated JSON endpoint | `controller` |
| `03-post-authorization` | only the author may edit a post | `voter` |

Every task except the control maps onto a skill that actually exists here. That sounds
obvious and was learned the hard way: the first run's other two tasks were built around
`symfony/lock` and Messenger, which **no skill in this repository mentions**. They could not
have measured anything, and reading their flat results as "the baseline is strong" was
wrong — they were null tests.

`01-lock` is kept deliberately as a control. No skill covers it, so it should show no
difference between the variants. If it ever does, the benchmark is measuring something other
than the skills — noise, ordering, or wishful thinking — and the other results are worth
less.

## Running it

```bash
bin/run 02-json-endpoint without-skills
bin/run 02-json-endpoint with-skills
bin/judge 02-json-endpoint
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

`results/` is not committed. Projects are built under `$TMPDIR/symfony-skills-benchmark`;
override with `BENCHMARK_WORKDIR`.

Agents tend to start the skeleton's `compose.yaml` for a database. `bin/run` shuts it down
afterwards, but check `docker ps` if a run is interrupted.
