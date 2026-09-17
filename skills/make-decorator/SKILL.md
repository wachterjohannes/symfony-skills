---
name: make-decorator
description: Use when an existing service's behavior must change without editing its class, such as adding caching, logging or a fallback around it.
version: 1.0.0
updated: 2026-09-17
symfony-versions: ">=7.4"
maker-bundle-versions: ">=1.68"
---

# Service decorator

```bash
symfony console make:decorator translator App\\Translation\\LoggingTranslator
```

Both values are arguments: the id of the service to decorate, then the class to create.
Non-interactively the id must be exact. An interactive run guesses near misses and offers
a choice, a non-interactive run fails instead. `symfony console debug:container` lists the
ids. `--priority` orders multiple decorators on the same service.

## What you get

A class marked `#[AsDecorator('<id>')]`, a constructor that receives the decorated
service through `#[AutowireDecorated]`, and one forwarding method per public method of the
decorated class. No configuration file changes: the attribute is the registration.

Change only the methods whose behavior should differ and leave the rest forwarding. A
decorator that reimplements everything is a replacement, and then the honest move is a
new service, not a decorator.

## When to decorate

Decorating earns its place when the class is not yours to edit: a vendor service, a
framework service, or a class other code depends on staying as it is. When you own the
class and can change it, change it. A decorator around your own service adds indirection
without buying anything.
