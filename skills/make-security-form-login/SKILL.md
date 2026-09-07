---
name: make-security-form-login
description: Use when the application needs a server-rendered login form.
version: 1.0.0
updated: 2026-09-07
symfony-versions: ">=6.4"
---

# Form login

```bash
symfony console make:security:form-login --controller-name=SecurityController --logout
```

That is usually enough: the maker reads `security.yaml` and fills in the firewall, the
user class and the username field itself when they are unambiguous. When a guess is
ambiguous it aborts naming the option to pass (`--firewall-name`, `--user-class`,
`--username-field`) rather than prompting. `--logout` is deliberate: the prompt defaults
to yes, the option is opt-in, so a script states what it wants.

A user class and provider must exist first; that is `make:user`, the step before this
one. Install the maker if it is missing:
`symfony composer require --dev symfony/maker-bundle`.

## What you get, and what to keep

A login controller with its Twig template, and a `form_login` entry under the firewall in
`security.yaml`. Keep the shape: the controller only renders the form and exposes the
last error, while the actual credential check runs inside the security system. A login
controller that starts reading the submitted password is rebuilding the authenticator by
hand. Leave CSRF protection on.

## What to change

The template is a starting point and looks it. Where the user lands after login and
logout is application policy, not something the maker can know.
