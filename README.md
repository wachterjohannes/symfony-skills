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
| [`components`](skills/components/SKILL.md) | check for a Symfony component before hand-rolling infrastructure |
| [`services`](skills/services/SKILL.md) | autowiring first: what needs no configuration, and the attributes for the rest |

**Maker wrappers** point at a MakerBundle command and cover what it cannot decide. They
carry no PHP templates, because rebuilding `make:*` as a prompt is worse than running it:

| Skill | Wraps |
|---|---|
| [`make-command`](skills/make-command/SKILL.md) | `make:command` |
| [`make-crud`](skills/make-crud/SKILL.md) | `make:crud` |
| [`make-entity`](skills/make-entity/SKILL.md) | `make:entity` |
| [`make-listener`](skills/make-listener/SKILL.md) | `make:listener` |
| [`make-test`](skills/make-test/SKILL.md) | `make:test` |
| [`make-controller`](skills/make-controller/SKILL.md) | `make:controller` |
| [`make-form`](skills/make-form/SKILL.md) | `make:form` |
| [`make-user`](skills/make-user/SKILL.md) | `make:user` |
| [`make-migration`](skills/make-migration/SKILL.md) | `make:migration` |
| [`make-voter`](skills/make-voter/SKILL.md) | `make:voter` |
| [`make-twig-extension`](skills/make-twig-extension/SKILL.md) | `make:twig-extension` |

### Which makers get a wrapper

A maker gets one only if every value it needs can be passed on the command line. Counting the
questions it asks is not that test — most of them merely fill an argument that could have been
supplied, which is why `make:form` and `make:user` were wrongly excluded for a while.

`make:entity` and `make:crud` were genuinely blocked and no longer are:
[maker-bundle#1810](https://github.com/symfony/maker-bundle/pull/1810) added `--field`,
[#1813](https://github.com/symfony/maker-bundle/pull/1813) added `--relation`, and
[#1814](https://github.com/symfony/maker-bundle/pull/1814) added `--controller-class` and
fixed the crash that made `make:crud` unusable under `--no-interaction`.

A second PR series is open for most of what remained:
[#1816](https://github.com/symfony/maker-bundle/pull/1816) (`make:security:custom`),
[#1817](https://github.com/symfony/maker-bundle/pull/1817) (`make:webhook`),
[#1818](https://github.com/symfony/maker-bundle/pull/1818) (`make:security:form-login`),
[#1819](https://github.com/symfony/maker-bundle/pull/1819) (`make:reset-password`) and
[#1820](https://github.com/symfony/maker-bundle/pull/1820) (`make:registration-form`) each
fix the crash under `--no-interaction` by moving the asked values into options. `make:auth`
is deprecated in favour of the `make:security:*` commands, so its share of the work lives
there. Skills for these makers follow once the PRs are merged — until then, still blocked
and still without a skill here, alongside `make:schedule`, which has no PR yet.

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
mechanism that reaches into your project — a skill you edited is yours. The repository
itself stays current the other way round: [`tools/update-skills`](tools/update-skills/)
watches the documentation pages each skill draws on, and a monthly workflow files any
drift as an issue.

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
