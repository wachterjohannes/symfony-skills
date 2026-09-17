---
name: make-command
description: Use when creating a console command in a Symfony project.
version: 2.0.0
updated: 2026-09-17
symfony-versions: ">=7.4"
maker-bundle-versions: ">=1.68"
---

# Console command

```bash
symfony console make:command app:send-reminders --argument recipient:string --option dry-run:bool
```

Install the bundle first if it is missing: `symfony composer require --dev symfony/maker-bundle`.

Declare every argument and option on the command line. `--argument name[?][:type]` and
`--option name[:type]` repeat once per parameter. A `?` marks an argument optional. Types
are `string`, `bool`, `int`, `float`, `array`, or the name of a backed enum. The default
type is `string` for arguments and `bool` for options. If you pass neither flag, a
non-interactive run generates sample parameters (`$arg`, `$enable`) that you would have to
delete, so declare the real ones up front.

## What you get, and what to keep

Since maker-bundle 1.68 the command is invokable. The generated class carries
`#[AsCommand]` and a single `__invoke()` method. Its parameters are the command's
interface: a `SymfonyStyle` instance, then one parameter per argument and option, each
declared with an `#[Argument]` or `#[Option]` attribute. There is no base class, no
`configure()`, no `execute()`. Do not add them back.

What remains yours:

- Inject dependencies through the constructor.
- Return `Command::SUCCESS` or `Command::FAILURE`. Never a bare integer, never nothing.
- Write output through `SymfonyStyle` (`$io->success()`, `$io->error()`, `$io->table()`)
  rather than `echo`, so the command behaves under `--quiet` and in a pipe.
- Keep the logic in a service and let `__invoke()` orchestrate. A command is an entry
  point, the same as a controller.

A command run from cron that must not overlap takes `symfony/lock`: inject `LockFactory`,
or use `LockableTrait` when the command only locks itself. A flag file or a database
column checked at start is not a lock.
