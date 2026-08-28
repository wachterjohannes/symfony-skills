---
name: migration
description: Use when the database schema needs to change in a Symfony project using Doctrine.
version: 1.0.0
updated: 2026-08-28
symfony-versions: ">=6.4"
---

# Schema changes

```bash
symfony console make:migration
symfony console doctrine:migrations:migrate
```

`make:migration` takes no arguments and asks nothing — it diffs the mapping against the
database and writes the migration. Requires
`doctrine/doctrine-migrations-bundle`.

## The rule

Every schema change goes through a migration. Not `doctrine:schema:update`, not hand-written
SQL against the database. The migration is the only artefact that can be reviewed, replayed
on another environment, and rolled back.

## Reading what was generated

Open the file before running it. The diff is derived from the mapping, so a renamed property
appears as a dropped column plus a new one — which loses data. Rewrite those cases by hand
inside the generated `up()` method.

`--formatted` makes the SQL readable when you intend to edit it.

## Test and production

`doctrine:migrations:migrate` runs the same way everywhere. For the test database, run it
with `--env=test`, and prefer it over recreating the schema so tests exercise the migrations
that production will run.
