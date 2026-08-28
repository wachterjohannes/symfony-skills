---
name: form
description: Use when building a server-rendered form in a Symfony project.
version: 1.0.0
updated: 2026-08-28
symfony-versions: ">=6.4"
---

# Form

```bash
symfony console make:form ProductType Product      # bound to an entity
symfony console make:form ContactType              # bound to nothing
```

Both the form class and the class it binds to are arguments, so the maker asks nothing when
you pass them. Install it first if it is missing:
`symfony composer require --dev symfony/maker-bundle`.

## Where things belong

**Validation constraints go on the object, not on the form fields.** A constraint attached
to a field only exists inside that one form; on the entity or DTO it applies everywhere the
object is used.

**Buttons go in the template, not in the form class.** A form used for both creating and
editing needs "Add new" in one place and "Save changes" in the other — the class should not
know which. Styling belongs in the template too. The exception is a form with several submit
buttons, where the controller has to tell them apart; define those in the controller.

**One action renders and processes.** Rendering and handling a submission are near-identical
in a controller, so splitting them across two actions duplicates the setup for nothing:

```php
$form = $this->createForm(ProductType::class, $product);
$form->handleRequest($request);

if ($form->isSubmitted() && $form->isValid()) {
    // persist, then redirect
}

return $this->render('product/edit.html.twig', ['form' => $form]);
```

## What the maker leaves you

A `FormType` with a `buildForm()` and a `configureOptions()`. Keep the `data_class` it set
when the form is bound. Field types are guessed from the mapping and are worth revisiting —
an entity property is not always the widget a user should see.
