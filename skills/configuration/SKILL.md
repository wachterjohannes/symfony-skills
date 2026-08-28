---
name: configuration
description: Use when deciding where a configuration value belongs — environment variable, parameter, class constant, or secret.
version: 1.0.0
updated: 2026-08-28
symfony-versions: ">=6.4"
---

# Symfony configuration

Four places, distinguished by what makes the value change.

| The value changes | Put it in                        | Example                        |
|-------------------|----------------------------------|--------------------------------|
| Per machine       | environment variable (`.env`)    | `DATABASE_URL`, `MAILER_DSN`   |
| Per environment   | parameter (`config/services.yaml`) | `app.notifications_email`    |
| Almost never      | class constant                   | `Post::ITEMS_PER_PAGE`         |
| Must stay secret  | the vault (`secrets:set`)        | API keys, passwords            |

## Environment variables

Infrastructure only — hosts, credentials, DSNs. Defaults live in `.env`, which is
committed. Real values live in `.env.local`, which is not. Read them with `%env(...)%`.

Application behaviour does not belong here.

## Parameters

Behaviour that differs per environment. Prefix everything with `app.` so it can never
collide with a parameter from Symfony or a bundle.

```yaml
# config/services.yaml
parameters:
    app.notifications_email: 'noreply@example.com'
    app.items_per_page: 10
```

## Class constants

Domain values that are intrinsic to the code rather than to a deployment. They work in
Twig, in Doctrine queries and in plain PHP without going through the container.

```php
final class Post
{
    public const ITEMS_PER_PAGE = 10;
    public const STATUS_DRAFT = 'draft';
}
```

## Secrets

```bash
symfony console secrets:set DATABASE_PASSWORD
```

The vault is encrypted and safe to commit. Each environment has its own.
