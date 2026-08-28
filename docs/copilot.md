# GitHub Copilot

Copilot reads instruction files from `.github/instructions/`, each with an `applyTo` glob,
plus `.github/copilot-instructions.md` which applies to everything.

## Install

For each skill, create `.github/instructions/<name>.instructions.md` containing the skill's
body, with this frontmatter:

```yaml
---
applyTo: "**"
---
```

Narrow the glob where a skill only concerns part of the tree:

```yaml
---
applyTo: "src/Controller/**"
---
```

## The always-on file

`.github/copilot-instructions.md` is loaded for every request, so it is the wrong place for
skill bodies. Put the index table from this repository's `AGENTS.md` there instead, if you
want one at all.

## Updating

Recopy the bodies.

## Removing

Delete the instruction files.
