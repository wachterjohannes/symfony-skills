# Symfony Skills

A foundational set of Symfony best practices, written as skills for AI coding agents, so
that generated Symfony code looks like Symfony code — idiomatic, using the framework's own
components instead of hand-rolled substitutes.

> **Prototype.** This is an experiment ahead of a discussion in the Symfony core team, not
> a finished product. See [PLANNING.md](PLANNING.md) for what is decided and what is not.

## The problem

Ask an agent to add a lock around a cron job and it writes one. Ask it to accept JSON and it
reaches for `json_decode()`. Symfony has `symfony/lock` and `#[MapRequestPayload]`, and the
agent had no reason to know that mattered.

These skills give it that reason.

## What is in here

Two kinds of skill, and the difference is the point.

**Background knowledge** applies whenever the agent touches Symfony code:

| Skill | Covers |
|---|---|
| [`cli-conventions`](skills/cli-conventions/SKILL.md) | `symfony console` over `bin/console`, and why the prefix matters |
| [`configuration`](skills/configuration/SKILL.md) | env vars vs. parameters vs. constants vs. secrets |
| [`templates`](skills/templates/SKILL.md) | Twig naming, fragments, AssetMapper |
| [`discover`](skills/discover/SKILL.md) | look it up in the project instead of recalling it |

**Maker wrappers** point at a MakerBundle command and cover what it cannot decide. They
carry no PHP templates, because rebuilding `make:*` as a prompt is worse than running it:

| Skill | Wraps |
|---|---|
| [`command`](skills/command/SKILL.md) | `make:command` |
| [`controller`](skills/controller/SKILL.md) | `make:controller` |
| [`migration`](skills/migration/SKILL.md) | `make:migration` |
| [`voter`](skills/voter/SKILL.md) | `make:voter` |
| [`twig-extension`](skills/twig-extension/SKILL.md) | `make:twig-extension` |

### Why there is no entity or form skill

A maker gets a wrapper only if it runs to completion without prompting. `make:entity` asks
for one property at a time in a loop and exposes no way to pass fields as arguments, and
`make:form`, `make:user`, `make:crud` and `make:auth` prompt too. Writing a PHP template
instead would be exactly the thing this repository argues against, so those skills are
absent until `make:entity` accepts fields on the command line — a contribution we intend to
make.

## Installing

Copy the folders. There is no install script.

| Agent | How |
|---|---|
| Claude Code | [docs/claude-code.md](docs/claude-code.md) |
| Codex | [docs/codex.md](docs/codex.md) |
| Cursor | [docs/cursor.md](docs/cursor.md) |
| GitHub Copilot | [docs/copilot.md](docs/copilot.md) |
| OpenCode | [docs/opencode.md](docs/opencode.md) |

Copy it in, adapt it when you need to. These are not a dependency, and there is no update
mechanism that reaches into your project — a skill you edited is yours.

## Relation to the framework-bundle `AGENTS.md`

[symfony/recipes#1563](https://github.com/symfony/recipes/pull/1563) adds an `AGENTS.md` to
new Symfony projects. Both artefacts stand on their own: that file works for someone who
never installs a skill, and these skills work in a project without it. Where they overlap,
**the recipe file wins** — a skill that contradicts it is a bug in the skill.

## Credits

Built on the skill set by [welcoMattic](https://github.com/welcoMattic), originally at
[welcoMattic/symfony-skills](https://github.com/welcoMattic/symfony-skills). That repository
is archived and its author allowed the reuse. The structure here diverges — folders instead
of a central metadata file, no install script, no code templates — but the starting point
was his.

## License

MIT. See [LICENSE](LICENSE).
