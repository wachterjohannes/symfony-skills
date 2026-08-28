---
name: cli-conventions
description: Use when running any command in a Symfony project — console commands, Composer, the local web server, or tests.
version: 1.0.0
updated: 2026-08-28
symfony-versions: ">=6.4"
---

# Symfony CLI conventions

Run the app with `symfony serve -d`; run commands with `symfony console ...` (or
`bin/console` if the Symfony CLI is not available).

`command -v symfony` tells you which case you are in.

## Prefixes

| Direct                  | With the Symfony CLI      |
|-------------------------|---------------------------|
| `php bin/console <cmd>` | `symfony console <cmd>`   |
| `composer <cmd>`        | `symfony composer <cmd>`  |
| `php -S localhost:8000` | `symfony serve -d`        |
| `php bin/phpunit`       | `symfony php bin/phpunit` |

The prefix matters because the Symfony CLI picks the PHP version configured for the
project, loads the `.env` files, terminates TLS locally, and exports the environment
variables of Docker services such as the database or mailer. Without it those values are
simply absent, and the failure looks like a configuration bug.

## Server

```bash
symfony serve -d       # start in the background
symfony server:stop
symfony server:log     # tail the logs
```

## Worth knowing

```bash
symfony check:requirements    # PHP version and extensions
symfony check:security        # known vulnerabilities in the lock file
symfony var:export --debug    # what the CLI injects into the environment
```
