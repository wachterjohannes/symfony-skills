---
name: discover
description: Use before relying on memory for a Symfony API, route, service or configuration value — look it up in the project instead.
version: 1.0.0
updated: 2026-08-28
symfony-versions: ">=6.4"
---

# Discover, don't guess

Framework APIs and attribute names change between major versions, and training data goes
stale quietly. The project itself is authoritative; ask it.

## What is installed

`composer.json` gives the Symfony and PHP versions. `symfony.lock` lists the recipes that
ran, which is the reliable signal for whether Doctrine, Twig, API Platform, Messenger or
Lock are actually present. Assume nothing that neither file confirms.

## What exists at runtime

```bash
symfony console about                        # versions, environment, paths
symfony console debug:router                 # every route
symfony console debug:container              # every service
symfony console debug:autowiring <name>      # what a type-hint resolves to
symfony console debug:config <bundle>        # effective configuration
symfony console config:dump-reference <bundle>   # every available option
```

## Before running it

```bash
symfony console lint:container
symfony console lint:twig templates/
symfony console lint:yaml config/
```

## When the console cannot answer

Read the installed source and its docblocks under `vendor/`. That is the code that will
actually run — more current than any recollection of it.

The documentation at <https://symfony.com/doc/current/> is the canonical reference; switch
it to the version in `composer.json` when the project is not on the current release.
