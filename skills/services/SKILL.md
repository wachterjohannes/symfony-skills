---
name: services
description: Use when creating a service class, wiring dependency injection, or about to edit services.yaml.
version: 1.0.0
updated: 2026-09-03
symfony-versions: ">=6.4"
---

# Services and dependency injection

The default `services.yaml` already registers every class in `src/` as a service,
autowired and autoconfigured. A new class is a service the moment the file exists. The
most common mistake is adding configuration for it anyway.

Training data is full of the old style: explicit service ids, `arguments:` lists,
`public: true`, tags wired by hand. None of that is current. If you are writing a
`services.yaml` entry, name the reason; the legitimate ones are a vendor class outside
`src/` and a per-environment override. Everything else has an attribute.

## What needs no configuration

- Constructor type-hints resolve themselves. An interface with exactly one implementation
  autowires like a class.
- `#[AsEventListener]`, `#[AsCommand]`, `#[AsMessageHandler]` and friends register the
  service where it is used. No tag entry anywhere.
- Services are private and stateless by default. Keep them that way: nothing fetches from
  the container, everything is injected. `ContainerInterface` in a constructor is the
  container pattern the defaults exist to prevent.

## When you do configure, do it in the class

```php
public function __construct(
    #[Autowire('%kernel.project_dir%')] private string $projectDir,
    #[AutowireIterator('app.exporter')] private iterable $exporters,
) {
}
```

- A scalar comes in via `#[Autowire('%parameter%')]` or `#[Autowire(env: 'NAME')]`. Where
  the value itself belongs is the `configuration` skill's question.
- Several implementations of one interface: `#[AutowireIterator]` or `#[AutowireLocator]`
  on the consumer, `#[AsTaggedItem]` on the implementations.
- One implementation should win the type-hint: `#[AsAlias]` on it.

Constructor injection is the default; reach for anything else only with a reason the code
can show.

## Checking

`symfony console debug:autowiring <type>` says what a type-hint resolves to, and
`debug:container <service>` shows the final definition. Ask, don't guess; the `discover`
skill lists the rest.
