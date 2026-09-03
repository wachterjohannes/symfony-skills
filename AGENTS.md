# Symfony skills

This project carries a set of Symfony skills under `skills/`. Each is a short document
covering one topic. Read a skill when its trigger applies; don't read them all up front.

| Skill | Read it when | File |
|---|---|---|
| `cli-conventions` | running any command — console, Composer, server, tests | `skills/cli-conventions/SKILL.md` |
| `configuration` | deciding where a configuration value belongs | `skills/configuration/SKILL.md` |
| `templates` | writing Twig, naming templates, adding CSS or JavaScript | `skills/templates/SKILL.md` |
| `discover` | about to rely on memory for a Symfony API, route or service | `skills/discover/SKILL.md` |
| `components` | about to hand-write a lock, queue, cache, HTTP client, mailer or scheduler, or add a library for one | `skills/components/SKILL.md` |
| `services` | creating a service class, wiring dependency injection, or editing services.yaml | `skills/services/SKILL.md` |
| `make-command` | creating a console command | `skills/make-command/SKILL.md` |
| `make-crud` | an entity needs list, show, create, edit and delete | `skills/make-crud/SKILL.md` |
| `make-entity` | creating a Doctrine entity, or adding fields to one | `skills/make-entity/SKILL.md` |
| `make-listener` | code should run on a framework or application event | `skills/make-listener/SKILL.md` |
| `make-test` | writing a test, or choosing its base class | `skills/make-test/SKILL.md` |
| `make-controller` | creating a controller or an HTTP endpoint | `skills/make-controller/SKILL.md` |
| `make-form` | building a server-rendered form | `skills/make-form/SKILL.md` |
| `make-user` | the project needs a security user class | `skills/make-user/SKILL.md` |
| `make-migration` | changing the database schema | `skills/make-migration/SKILL.md` |
| `make-voter` | access depends on who is asking, not just on a role | `skills/make-voter/SKILL.md` |
| `make-twig-extension` | a template needs a filter or function Twig lacks | `skills/make-twig-extension/SKILL.md` |

If this project also has a Symfony `AGENTS.md` from the framework-bundle recipe, that file
takes precedence. These skills deepen it; they never contradict it.
