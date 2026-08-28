- Uses `symfony/lock` — either a `LockFactory` obtained through dependency injection or
  `LockableTrait`, which wraps the same component.
- Does not hand-roll mutual exclusion — no lock file written by the command itself, no
  database flag column, no PID check, no `flock()` call.
- Adds the component with `composer require symfony/lock` — the skeleton does not ship it,
  so the recipe artifacts (`config/packages/lock.yaml`, the `LOCK_DSN` entry in `.env`) are
  present and `config/bundles.php` was not edited by hand.
- Releases the lock, or relies on the component doing so.
