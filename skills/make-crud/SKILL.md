---
name: make-crud
description: Use when an entity needs the full set of list, show, create, edit and delete actions.
version: 1.0.0
updated: 2026-08-31
symfony-versions: ">=6.4"
---

# CRUD for an entity

```bash
symfony console make:crud BlogPost --controller-class=BlogPostController
```

The entity is the argument and `--controller-class` names the controller; without it the name
is derived from the entity. The maker also writes the form type and the Twig templates.
Install it first if it is missing: `symfony composer require --dev symfony/maker-bundle`.

## Before running it

`make:crud` generates a lot — a controller with five actions, a form type, five templates. On
an entity that only needs a list and a detail page, most of that is dead code someone has to
read later. Generate it when the entity really is edited through a full admin-style interface,
and write the one or two actions by hand when it is not.

## What to keep

The generated controller extends `AbstractController` and routes through attributes, with
route names prefixed by the entity. Actions take the entity as a type-hinted argument and let
the value resolver load it, which is why a missing record 404s without any code.

Create and edit each render and process the form in a single action. That is the convention,
not an accident of the generator.

The delete action checks a CSRF token. Removing that check because it is inconvenient makes
every listing page a deletion link away from a cross-site request.

## What to change

The templates are a starting point and look it. Field types in the generated form are guessed
from the mapping — an entity property is not always the widget a user should see.
