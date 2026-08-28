- Uses `symfony/lock` — a `LockFactory` obtained through dependency injection.
- Does not hand-roll mutual exclusion — no lock file written by the command itself, no
  database flag column, no PID check, no `flock()` call.
- Adds the component with `composer require symfony/lock` rather than editing
  `config/bundles.php` by hand.
- Releases the lock, or relies on the component doing so.
