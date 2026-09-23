---
name: make-ci
description: Use when a project needs a CI pipeline or workflow for GitHub Actions or GitLab CI, such as running tests and linters on every push or pull request.
version: 1.0.0
updated: 2026-09-23
symfony-versions: ">=7.4"
maker-bundle-versions: ">=1.69"
---

# CI configuration

```bash
symfony console make:ci --platform=github-actions --php-version=8.4
```

`--platform` takes `github-actions` or `gitlab-ci` and is required non-interactively. The
maker detects the rest: installed tools from `composer.json`, translation formats from
`translations/`, the default branch from git, and the database engine from `DATABASE_URL`
in `.env`. Two detections deserve overriding as a rule, not an exception:

- `--php-version` (format `major.minor`). The detection takes the first version in the
  `php` constraint of `composer.json`, so `>=8.2` yields 8.2 even when production runs
  8.4. Pass the real version.
- `--branch`. The detection only works in a clone with a remote. A fresh repository
  silently falls back to `main`.

`--database` takes `postgres`, `mysql`, `mariadb`, `sqlite` or `none` and only has an
effect when Doctrine is installed. A missing or non-standard `DATABASE_URL` silently
falls back to postgres, so pass the option whenever the DSN is unusual.

## What you get

`.github/workflows/ci.yaml`, plus a `dependabot.yml` for composer and github-actions
updates when none exists. For GitLab, `.gitlab-ci.yml`. The lint job is always there.
Test, PHP CS Fixer, Twig CS Fixer and PHPStan jobs exist only when the project has the
tool.

## What is yours

- Only the exact target paths are protected: the maker refuses to overwrite
  `.github/workflows/ci.yaml` and `.gitlab-ci.yml`, nothing else. A project with CI under
  another file name gets a second, parallel workflow. Check what exists before running.
- The actions are pinned by SHA with the version as a comment. Keep them pinned. Do not
  simplify a pin to a tag.
- Project-specific steps such as deployments, coverage uploads or custom composer scripts
  are yours to add. A tool that lives outside `composer.json`, in a `tools/` directory or
  as a phar, gets no job either. Add it by hand.
