# Updating the skills

```bash
bin/diff     # what changed in the documentation since the snapshot
bin/fetch    # refresh the snapshot
```

`sources.json` lists the documentation files that feed the skills, and which skill each one
feeds. Add a source there rather than in a script.

`bin/diff` fetches into a temporary directory and compares it against `snapshot/`, which is
committed. It prints only the changed hunks, grouped by affected skill, and never decides
whether a change matters.

That decision is a prompt, not a script: copy `SKILL.md` into `.claude/skills/update-skills/`
(or the equivalent for your agent) and run it against the output of `bin/diff`.

## Prerequisites

`curl` and `jq`.
