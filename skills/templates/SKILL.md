---
name: templates
description: Use when creating or editing Twig templates, naming template files, or adding CSS and JavaScript to a Symfony project.
version: 1.0.0
updated: 2026-08-28
symfony-versions: ">=6.4"
---

# Twig templates

## Naming

Everything is snake_case — files, directories and variables.

```
templates/product/index.html.twig        not  templates/Product/Index.html.twig
templates/user_profile/edit.html.twig    not  templates/UserProfile/Edit.html.twig
```

Fragments meant to be included or embedded get a leading underscore, so a full page and a
partial are distinguishable at a glance:

```
templates/product/_card.html.twig
templates/layout/_navigation.html.twig
```

Variables passed from a controller follow the same rule:

```php
return $this->render('product/show.html.twig', [
    'product_list' => $products,
    'is_published' => true,
]);
```

## URLs

Generate them with `path()` and `url()`. A hard-coded URL in a template survives exactly
until someone changes a route.

## Assets

AssetMapper, not Webpack or Encore. It serves modern JavaScript and CSS without a build
step. Encore remains valid in projects that already use it — this is about what to reach
for in a project that has neither.
