---
name: twig-extension
description: Use when a template needs a filter or function that Twig does not provide.
version: 1.0.0
updated: 2026-08-28
symfony-versions: ">=6.4"
---

# Twig extension

```bash
symfony console make:twig-extension AppExtension
```

The maker takes the class name as an argument and asks nothing further. Install it first if
it is missing: `symfony composer require --dev symfony/maker-bundle`.

## Before writing one

Check that Twig or Symfony does not already cover it. `format_currency`, `format_datetime`,
`u` from the String component, and the `serializer` extension between them cover most of
what people write extensions for. `symfony console debug:twig` lists everything currently
available, filters and functions included.

## What to keep

The maker generates an extension plus a separate runtime class. Keep that split: the
extension only declares names, the runtime holds the implementation and its dependencies, so
nothing is instantiated for a template that never calls it.

Presentation logic only. A filter that queries the database has moved business logic into
the template layer.
