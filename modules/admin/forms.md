---
description: Admin form chrome — fieldsets, tables, floating save, tips, and required markers.
---

# Forms

Admin forms share a small set of conventions on top of the [common form helpers](../../../key-concepts/form-fields.md). This page covers the chrome; the helpers themselves live in `nails/common`.

## Fieldsets

A `<fieldset>` with a `<legend>` is the card around a group of fields. [DefaultController](controllers/default-controller.md) edit screens put each model fieldset in its own [tab](javascript/tabs.md); `Helper::tabs()` wraps loose tab content in a legend-less fieldset so it still gets card chrome.

To let the user fold a secondary group away, opt in with [`data-collapse`](javascript/collapsible-fieldsets.md).

## Floating save bar

`Helper::floatingControls()` renders the sticky save bar at the bottom of an edit form. DefaultController already does this. In a custom view:

```php
echo \Nails\Admin\Helper::floatingControls([
    'save' => [
        'text' => 'Save Changes',
    ],
    'unsaved_changes' => true,
    'notes' => [
        'enabled'  => true,
        'model'    => 'Book',
        'provider' => 'app',
    ],
    'html' => [
        'left'   => '',
        'center' => '',
        'right'  => '',
    ],
]);
```

`unsaved_changes` opts the enclosing form into the [Unsaved Changes](javascript/unsaved-changes.md) plugin (a chip beside Save, plus `beforeunload` while dirty). You can also put `data-unsaved-changes` on the `<form>` yourself. [DefaultController](controllers/default-controller.md) edit screens enable this by default.

A [screen-tab](javascript/screen-tabs.md) panel with `screen-tabs__panel--no-save` hides the bar while that workspace is active.

## Tables

Admin tables are borderless by default. Do **not** add `.table-bordered` — that class still draws a grid if you use it, but vendor views have dropped it.

Typical classes:

| Class            | Role |
| ---------------- | ---- |
| `table table-striped table-hover table-responsive` | Standard index / form table |
| `table-rounded`  | Opt-in radius on the corner cells. Fieldset cards already have a radius, so flush fieldset tables ignore this class. |
| `table-sticky`   | Sticky header under the admin topbar |

Cells are vertically centred. `tfoot` is white so [dynamic-table](javascript/dynamic-table.md) add-row buttons sit on a clean row. Put “+ Add Item” in `<tfoot>`, not under the table.

## Tips and required

Short guidance belongs in `tip` (question-mark beside the label). Longer copy, HTML, and alerts belong in `info`. See [Form fields](../../../key-concepts/form-fields.md).

`required => true` paints an asterisk with an accessible name via `Field::requiredMarker()`. It does **not** set HTML5 `required`, so draft saves are not blocked by the browser.

Readonly fields show a padlock on the control, not on the helper text.
