# Cursor

Cursor reads rule files from `.cursor/rules/`, each an `.mdc` file with frontmatter. The
skills map onto it directly — the body transfers unchanged, only the frontmatter differs.

## Install

For each skill, create `.cursor/rules/<name>.mdc` containing the skill's body, with this
frontmatter:

```yaml
---
description: <the description from the skill's frontmatter>
alwaysApply: false
---
```

`description` is what Cursor matches against, so keep the wording from the skill — it is
already written as a trigger.

Scope a rule to the files it concerns where that makes sense:

```yaml
globs: ["src/Controller/**"]
```

Leave `globs` off for the background skills; they are not tied to a directory.

## Updating

Recopy the bodies. The frontmatter you wrote stays as is.

## Removing

Delete the `.mdc` files.
