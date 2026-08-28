# Codex

Codex has no skill system. It reads `AGENTS.md` from the project root, so the skills go in
as files plus an index that tells Codex when to open them.

## Install

```bash
mkdir -p .agents
cp -R /path/to/symfony-skills/skills .agents/skills
```

Then copy the table from this repository's `AGENTS.md` into your project's `AGENTS.md`, and
adjust the paths to where you put the folders (`.agents/skills/...`).

The index is around thirty lines. That is deliberate: Codex loads `AGENTS.md` on every turn,
so the file holds triggers and paths, never the skill bodies.

## If the project already has a Symfony `AGENTS.md`

Keep it and append the table. The recipe's content stays first and takes precedence; the
skills deepen it.

## Updating

Copy the folders again and re-check the table against `skills/` — a new skill needs a new
row.

## Removing

Delete the folders and the table.
