---
name: make-test
description: Use when writing a test, or choosing which base class a test should extend.
version: 1.0.0
updated: 2026-08-31
symfony-versions: ">=6.4"
---

# Tests

```bash
symfony console make:test TestCase InvoiceCalculatorTest
symfony console make:test KernelTestCase InvoiceRepositoryTest
symfony console make:test WebTestCase CheckoutControllerTest
```

Both the type and the name are arguments, so nothing is asked. `symfony composer require --dev
symfony/test-pack` if the project has no test setup yet.

## Picking the base class

The choice is about how much of the framework the test needs, and each step up costs time.

| Base class | For |
|---|---|
| `TestCase` | plain PHPUnit — a class with no framework dependency |
| `KernelTestCase` | anything that needs a service from the container |
| `WebTestCase` | an HTTP request and its response, without JavaScript |
| `ApiTestCase` | API-oriented scenarios, with API Platform |
| `PantherTestCase` | a real browser and a real server, for JavaScript |

Reaching for `KernelTestCase` where `TestCase` would do boots the kernel for every test and
buys nothing.

## What a test is for

A feature is not done until something exercises it the way a caller would — an HTTP request
for a controller, a service call for a service. A test asserting that nothing threw is not
that.

Run them with `symfony php bin/phpunit`, falling back to `vendor/bin/phpunit`.

For anything touching the database, run the migrations against the test database rather than
recreating the schema, so the tests exercise what production will run.
