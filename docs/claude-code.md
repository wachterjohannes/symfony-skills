# Claude Code

Claude Code has a native skill system. Copy the folders in and it reads the frontmatter of
each skill, pulling the body in only when a skill applies.

## Per project

```bash
mkdir -p .claude/skills
cp -R /path/to/symfony-skills/skills/* .claude/skills/
```

## For every project

```bash
mkdir -p ~/.claude/skills
cp -R /path/to/symfony-skills/skills/* ~/.claude/skills/
```

Project skills win over personal ones when both define the same name, so a project can
override a skill by keeping its own copy.

`AGENTS.md` is not needed here — Claude Code finds the skills on its own.

## Updating

Copy again. If you edited a skill, keep your version: there is no merge, and your copy is
the one that matches your project.

## Removing

Delete the folders you copied.
