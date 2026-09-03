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

Nobody has to run it by hand: the monthly workflow
[`docs-drift.yml`](../../.github/workflows/docs-drift.yml) runs the same diff and files any
drift as an issue labelled `docs-drift`, hunks and affected skills included. While that
issue stays open, later runs comment on it instead of opening a second one.

The decision what a hunk means is a prompt, not a script: copy `SKILL.md` into
`.claude/skills/update-skills/` (or the equivalent for your agent) and run it against the
issue body, or against the output of `bin/diff` locally. The workflow never edits a skill
and never closes the issue; both stay with whoever judges the drift.

## Prerequisites

`curl` and `jq`.
