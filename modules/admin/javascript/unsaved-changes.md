---
description: Warn when an admin edit form has unsaved changes.
---

# Unsaved Changes

This plugin fingerprints an opted-in form, then shows an “Unsaved changes” chip to the right of the floating Save button when the current fields differ from that snapshot. Leaving the page while dirty also triggers the browser’s `beforeunload` prompt.

{% hint style="info" %}
The check is generic: it reads successful form fields (`input`, `select`, `textarea`). CSRF token fields are ignored. It does not hook editor APIs; if a control writes into a field without an event, the poll still picks it up once the field value changes.
{% endhint %}

## Opting in

Add `data-unsaved-changes` to the `<form>`. Set it to `false` to opt out of a parent that already opted in.

```php
echo form_open(null, 'data-unsaved-changes');
echo \Nails\Admin\Helper::floatingControls();
echo form_close();
```

Alternatively, pass `unsaved_changes` to the [floating save bar](../forms.md#floating-save-bar). That stamps the same attribute on `.admin-floating-controls`; the plugin binds the closest form.

```php
echo \Nails\Admin\Helper::floatingControls([
    'unsaved_changes' => true,
]);
```

[DefaultController](../controllers/default-controller.md) edit screens are on by default (`EDIT_UNSAVED_CHANGES_ENABLED`). Set the constant to `false` on a controller to opt out.

## Behaviour

1. After other admin plugins have had a chance to initialise, the plugin snapshots named field values.
2. It compares again on `input`/`change` and on a short poll.
3. When dirty, the chip appears beside Save and the form gets `.is-dirty`.
4. Submit clears the notice and skips `beforeunload`.
