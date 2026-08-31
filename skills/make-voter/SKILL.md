---
name: make-voter
description: Use when access to an object depends on who is asking — ownership, state, or role combined with data.
version: 1.0.0
updated: 2026-08-28
symfony-versions: ">=6.4"
---

# Security voter

```bash
symfony console make:voter PostVoter
```

The maker takes the class name as an argument and asks nothing further. Install it first if
it is missing: `symfony composer require --dev symfony/maker-bundle`.

## When a voter is the right answer

A role answers "who is this user". A voter answers "may this user do this to that object".
Ownership, workflow state, or a rule combining a role with data belongs in a voter — not in
an `if` inside a controller, where it cannot be reused and cannot be tested on its own.

## What to keep

The generated class implements `supports()` and `voteOnAttribute()`. Declare the supported
attributes as class constants (`PostVoter::EDIT`) so callers cannot typo a string.

Call it through the framework:

```php
#[IsGranted(PostVoter::EDIT, subject: 'post')]
```

or `$this->denyAccessUnlessGranted(PostVoter::EDIT, $post)` when the decision happens mid-action.

A voter is an ordinary service — inject what it needs through the constructor, and unit-test
it directly.
