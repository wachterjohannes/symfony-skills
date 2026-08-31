---
name: make-entity
description: Use when creating a Doctrine entity or adding fields and relations to one.
version: 1.0.0
updated: 2026-08-31
symfony-versions: ">=6.4"
---

# Doctrine entity

```bash
symfony console make:entity BlogPost \
    --field=title:string:100 \
    --field=body:text \
    --field=publishedAt \
    --field=price:decimal:10:2 \
    --relation=author:ManyToOne:User
```

Fields are `name[:type[:length]][?]`, relations are `name:type:target[?]`, and a trailing `?`
makes either nullable. Both options repeat, once per field. Leaving the type off falls back to
the same guess the interactive mode makes, so `--field=publishedAt` becomes a
`datetime_immutable` and `--field=isActive` a boolean.

Enums still need the interactive mode. Install the maker first if it is missing:
`symfony composer require --dev symfony/maker-bundle`.

## What to keep

Doctrine mapping goes in attributes — `#[ORM\Entity]`, `#[ORM\Column]`, `#[ORM\ManyToOne]` —
never in YAML or XML. The maker does this; the point is not to undo it.

Dates are immutable. `datetime_immutable` and `\DateTimeImmutable`, not `datetime`, so a value
handed to something else cannot be changed underneath you.

Every entity gets a repository, and queries live there rather than in a controller or a
service. The maker generates one.

## Where the entity ends and the domain begins

Validation constraints belong on the entity, not on the form that happens to edit it —
attached to a form field they exist only inside that form.

Domain values that barely change are class constants on the entity (`Post::STATUS_DRAFT`),
usable from Twig, from a query, and from PHP without touching the container.

Getters and setters are generated for you. Adding behaviour beyond them is fine; an entity
that only carries data and has all its logic elsewhere is a choice, not a rule.

## After changing the mapping

`make:migration`, then run it. Never `doctrine:schema:update`.
