---
name: components
description: Use before hand-writing infrastructure such as a lock, a queue, a cache, an HTTP client, mail, scheduling or rate limiting, or before adding a third-party library for one of these.
version: 1.0.0
updated: 2026-09-01
symfony-versions: ">=6.4"
---

# Reach for the component

Before hand-writing infrastructure, or adding a third-party library for it, check whether
a Symfony component covers it. It usually does. The check costs one look; the hand-rolled
version costs whoever maintains it.

## The map

| The task needs | The component |
|---|---|
| a job that must not run twice at once | `symfony/lock`: `LockFactory`, or `LockableTrait` in a command |
| work moved off the request | `symfony/messenger`: a message class, `#[AsMessageHandler]`, an async transport |
| recurring jobs defined in PHP | `symfony/scheduler` |
| calling an HTTP API | `symfony/http-client`: `HttpClientInterface`, one scoped client per API |
| caching a computed value | `symfony/cache`: `CacheInterface::get()` with a callback |
| sending mail | `symfony/mailer`: `MailerInterface`, an `Email` object |
| throttling logins or API calls | `symfony/rate-limiter` |
| a status field with rules about transitions | `symfony/workflow` |
| unique identifiers | `symfony/uid`: `Uuid` or `Ulid` |
| time you need to control in tests | `symfony/clock`: `ClockInterface`, `ClockAwareTrait` |

## Using one

Check `symfony.lock` first; the project may already have it. If not:

```bash
symfony composer require symfony/lock
```

The Flex recipe wires the default configuration. Do not copy configuration by hand from a
blog post when a recipe exists.

This is not a rule to use every component. It is an order of operations: look first, write
code second. A `while` loop polling a database column is not a queue, a static array is
not a cache, and a timestamp column checked before sending is not a lock.
