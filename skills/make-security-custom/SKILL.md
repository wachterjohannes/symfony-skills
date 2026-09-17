---
name: make-security-custom
description: Use when authentication needs a custom authenticator because no built-in mechanism — form login, JSON login, access tokens, HTTP basic — fits.
version: 1.0.0
updated: 2026-09-07
symfony-versions: ">=6.4"
maker-bundle-versions: ">=1.68"
---

# Custom authenticator

```bash
symfony console make:security:custom ApiTokenAuthenticator
```

The class name is the argument; the class lands in `App\Security` with an
`Authenticator` suffix. The maker installs `symfony/security-bundle` itself when
`security.yaml` is missing, because the recipe is what writes that file.

## Before running it

A custom authenticator is the last resort. `form_login`, `json_login`, `access_token`
and `http_basic` cover the common mechanisms, and authorization is voters and
`#[IsGranted]`, not an authenticator. Reach here only when the mechanism itself is
nonstandard.

## What you get

A skeleton with the decisions stubbed out: `supports()` says whether this authenticator
handles the request, `authenticate()` builds the `Passport` (a `UserBadge`, and a
`SelfValidatingPassport` for token-style credentials), and the success and failure
handlers return the responses. That logic is yours; the maker cannot know it.

## Check the security.yaml diff

The maker registers the class under the `main` firewall's `custom_authenticators`, and
it sets that list rather than appending to it. An authenticator already registered there
is replaced, and `main` may not be the firewall you meant. Read the `security.yaml` diff
before committing.
