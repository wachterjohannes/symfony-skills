---
name: update-skills
description: Use when refreshing the Symfony skills against the current documentation, after `bin/diff` reports changes.
version: 1.0.0
updated: 2026-08-28
symfony-versions: "n/a"
---

# Updating the skills

The mechanical half is already done for you. `bin/fetch` downloads every configured source
into `snapshot/`, and `bin/diff` prints the hunks that moved since the committed snapshot,
each labelled with the skills that source feeds. Neither script decides anything.

Your job is the part a script cannot do: deciding whether a documentation change means a
skill is now wrong.

## What to do with a hunk

Most documentation changes are irrelevant here — a new example, a reworded paragraph, a
typo. A hunk matters when it changes what a developer should *do*:

- a recommendation reversed, or a new one added
- an API, attribute or command renamed
- something the skill presents as current is now discouraged
- a component now covers a case the skill still tells the agent to hand-write

Update the affected skill when one of those holds. Leave it alone otherwise — an unchanged
skill is a valid outcome, and the most common one.

## Constraints

Keep the existing structure. This is an update, not a rewrite: same file, same headings,
same order. A restructuring buries the actual change in noise and makes the diff
unreviewable.

Adding a new skill is allowed when the documentation introduces a topic that none of the
existing ones cover. Adding one to hold a paragraph that fits an existing skill is not.

Every skill stays subject to the writing rules in `PLANNING.md`: no numbered steps, no
defensive phrasing, no nested conditionals. If the update wants those, it belongs in
tooling instead.

When a skill changes, bump its `version` and set `updated` to today. Leave
`symfony-versions` alone unless the change is genuinely version-specific.

## Finishing

Refresh the snapshot with `bin/fetch` so the next run diffs against what you just read, and
open a pull request whose description says which documentation change drove which skill
edit. A reviewer should be able to check your judgement without re-reading the docs
themselves.

If a `docs-drift` issue triggered this, close it once the pull request is merged, stating
the outcome — which skills changed, or that none had to. The workflow only reports; it
never closes what it opened.
