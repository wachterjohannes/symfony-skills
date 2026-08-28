---
name: user
description: Use when a Symfony project needs a security user class — login, authentication, or stored accounts.
version: 1.0.0
updated: 2026-08-28
symfony-versions: ">=6.4"
---

# Security user

```bash
symfony console make:user User \
    --is-entity \
    --identity-property-name=email \
    --with-password
```

Every question the maker asks has an option, so passing all three makes it run without
prompting. Drop `--is-entity` for a user that does not live in the database, and
`--with-password` when another system checks credentials — a single sign-on server, for
instance.

Install the maker first if it is missing:
`symfony composer require --dev symfony/maker-bundle`.

## What it does beyond the class

The maker implements `UserInterface` and, with `--with-password`,
`PasswordAuthenticatedUserInterface`, and it wires the provider and the password hasher into
`config/packages/security.yaml`. That edit is the part worth reading afterwards — it is
security configuration, and it is the file everything else about authentication hangs off.

## What to keep

The identity property is what a user types to log in and what the provider looks up. It has
to be unique in the database; the maker adds the constraint, and removing it breaks the
lookup rather than merely allowing duplicates.

Roles are a `json` column holding strings. `getRoles()` always returns `ROLE_USER` in
addition to whatever is stored — leave that in, the security layer assumes it.

Never hash a password by hand. `UserPasswordHasherInterface` reads the algorithm from
`security.yaml`, so it stays correct when that configuration changes.
