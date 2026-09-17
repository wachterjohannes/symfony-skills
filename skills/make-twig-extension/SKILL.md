---
name: make-twig-extension
description: Use when a template needs a filter or function that Twig does not provide.
version: 2.0.0
updated: 2026-09-17
symfony-versions: ">=7.4"
maker-bundle-versions: ">=1.68"
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

Since maker-bundle 1.68 the extension is a single class. Each filter or function is a
method marked `#[AsTwigFilter('name')]` or `#[AsTwigFunction('name')]`. There is no
`getFilters()` list, no separate runtime class, and nothing to register: the attributes do
all of it, and Twig still instantiates the class only when a template calls one of its
names. Do not reintroduce the old `AbstractExtension` shape.

A filter that produces safe HTML declares it, otherwise its output gets escaped:
`#[AsTwigFilter('name', isSafe: ['html'])]`.

Presentation logic only. A filter that queries the database has moved business logic into
the template layer.
