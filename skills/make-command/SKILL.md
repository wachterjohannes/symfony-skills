---
name: make-command
description: Use when creating a console command in a Symfony project.
version: 1.0.0
updated: 2026-08-28
symfony-versions: ">=6.4"
---

# Console command

```bash
symfony console make:command app:send-reminders
```

The maker takes the command name as an argument and asks nothing further. Install it first
if it is missing: `symfony composer require --dev symfony/maker-bundle`.

## What you get, and what to keep

The generated class carries `#[AsCommand]`, extends `Command`, and hands you a
`SymfonyStyle` instance. Leave all three alone — they are the convention.

What remains yours:

- Inject dependencies through the constructor, and call `parent::__construct()`.
- Return `Command::SUCCESS` or `Command::FAILURE`. Never a bare integer, never nothing.
- Write output through `SymfonyStyle` (`$io->success()`, `$io->error()`, `$io->table()`)
  rather than `echo`, so the command behaves under `--quiet` and in a pipe.
- Keep the logic in a service and let `execute()` orchestrate. A command is an entry point,
  the same as a controller.
