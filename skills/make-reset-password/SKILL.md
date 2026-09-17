---
name: make-reset-password
description: Use when users need a forgot-password flow that emails a reset link.
version: 1.0.0
updated: 2026-09-11
symfony-versions: ">=6.4"
maker-bundle-versions: ">=1.68"
---

# Password reset

```bash
symfony console make:reset-password \
    --from-email-address=noreply@example.com --from-email-name="Acme" \
    --success-redirect-route=app_login
```

The two email values are required in a script: they have nothing to be derived from. The
rest is usually guessed silently from `security.yaml` and the user class; when a guess is
ambiguous the maker aborts naming the option to pass (`--user-class`, `--email-field`,
`--email-getter`, `--password-setter`). Pass `--success-redirect-route` deliberately: the
default is `app_home`, which is a guess about your route names, not a fact.

A user class must exist first (`make:user`). Install the maker if it is missing:
`symfony composer require --dev symfony/maker-bundle`.

## What you get

The flow comes from `symfonycasts/reset-password-bundle`, which the maker installs: a
controller with the request, check-email and reset actions, two form types, the Twig
templates, the reset email, and a `ResetPasswordRequest` entity holding the hashed
tokens. That entity changes the schema, so a migration follows (`make:migration`).

## What to check

- The emails only leave the application when a mailer transport is configured; without a
  real `MAILER_DSN` the flow dead-ends silently in dev.
- Keep the bundle's behaviour of answering the request page identically whether or not
  the email exists. Making the response say "no such account" turns the form into an
  account-enumeration oracle.
