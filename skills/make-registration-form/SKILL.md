---
name: make-registration-form
description: Use when the application needs self-service user registration.
version: 1.0.0
updated: 2026-09-11
symfony-versions: ">=6.4"
---

# Registration form

```bash
symfony console make:registration-form --unique-entity --auto-login \
    --redirect-route=app_home
symfony console make:registration-form --unique-entity \
    --verify-email --from-email-address=noreply@example.com --from-email-name="Acme"
```

Three flags are opt-in on the command line where the prompt would default to yes, so a
script states each one it wants: `--unique-entity` (a `UniqueEntity` constraint on the
user class — this is what prevents duplicate accounts), `--verify-email` (confirmation
mail before the account counts), `--auto-login` (sign the user in right after
registering; needs `--authenticator` when more than one exists). The identity fields
(`--user-class`, `--username-field`, `--password-field`, the getters) are guessed from
`security.yaml` and the class; an ambiguous guess aborts naming the option.

A user class must exist first (`make:user`). Install the maker if it is missing:
`symfony composer require --dev symfony/maker-bundle`.

## What you get

A registration controller, the form type with the plain-password field mapped to
nothing, and the template. With `--verify-email` the maker installs
`symfonycasts/verify-email-bundle` and adds the signed-confirmation route and email;
that path needs a configured mailer and the two `--from-email-*` values.

## What to check

- The password is hashed through the `UserPasswordHasherInterface` in the controller;
  the plain password never lands on the entity. Keep that shape.
- `--verify-email` without a real `MAILER_DSN` produces accounts nobody can activate.
- `--unique-entity` changed the user class, so the constraint shows up in its diff, not
  only in the controller.
