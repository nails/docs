---
description: Searchable Select2 dropdowns in admin.
---

# Select

Admin upgrades any visible `<select class="select2">` to a searchable Select2 3.5 dropdown. Fields inside [DefaultController](../controllers/default-controller.md) dropdowns, index filters, and `form_field_dropdown()` with `class => 'select2'` all pick this up.

```php
echo form_field_dropdown([
    'key'     => 'status',
    'label'   => 'Status',
    'class'   => 'select2',
    'options' => $aStatuses,
    'data'    => [
        'placeholder' => 'Choose a status',
        'clearable'   => 'true',
    ],
]);
```

## Options

Set these as `data-` attributes on the `<select>`:

| Attribute     | Description                                              | Default              |
| ------------- | -------------------------------------------------------- | -------------------- |
| `multiple`    | Allow more than one value                                | off                  |
| `clearable`   | Show a clear control                                     | off                  |
| `placeholder` | Placeholder text                                         | `Search for an item` |

Inside a `.field` the widget is 100% wide. Index search/filter selects stay compact.

## Hidden tabs

Select2 only binds `:visible` selects. Opening a [tab](tabs.md) or [screen tab](screen-tabs.md) calls `refreshUi()`, which instantiates dropdowns that were in a hidden panel.

The open menu is pinned to the field with viewport coordinates so it stays on screen inside overflow containers and after a panel that was `display: none` is shown.

## Searcher

For dropdowns populated from a CRUD API, use [Searcher](searcher.md) (`js-searcher`) instead of a static `<select>`.
