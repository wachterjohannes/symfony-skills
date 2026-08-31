---
name: make-listener
description: Use when code should run in response to a framework or application event.
version: 1.0.0
updated: 2026-08-31
symfony-versions: ">=6.4"
---

# Event listener

```bash
symfony console make:listener ExceptionListener kernel.exception
symfony console make:listener RequestSubscriber kernel.request
```

The suffix decides what is generated: a name ending in `Listener` produces a listener, one
ending in `Subscriber` a subscriber. Both the name and the event are arguments, so nothing is
asked. Install the maker first if it is missing:
`symfony composer require --dev symfony/maker-bundle`.

## Listener or subscriber

A listener declares one method with `#[AsEventListener]` and says in the attribute which event
and at what priority. A subscriber declares `getSubscribedEvents()` and carries that knowledge
in the class itself.

Prefer the listener. The attribute keeps the wiring next to the method it wires, and a class
handling several related events is the case where a subscriber earns its extra indirection.

## What to keep

The generated class is an ordinary service — inject what it needs through the constructor.

Priority matters when several listeners touch the same event, and the default of `0` is
usually right. `symfony console debug:event-dispatcher <event>` shows the order that actually
applies, which is worth checking before guessing at a number.

## Where a listener is the wrong answer

An event listener runs on every dispatch of that event, application-wide. A rule that applies
to one controller belongs in that controller, and a decision about access belongs in a voter.
Reach for a listener when the behaviour is genuinely cross-cutting.
