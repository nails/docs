---
description: Native boolean switches in admin forms.
---

# Toggles

Admin paints an iOS-style switch over a checkbox. The checkbox is still the form control; the switch is what the user sees and clicks.

`form_field_boolean()` emits the markup. You do not need to instantiate anything:

```php
echo form_field_boolean([
    'key'      => 'is_published',
    'label'    => 'Published',
    'default'  => true,
    'text_on'  => 'Published',
    'text_off' => 'Draft',
]);
```

The helper outputs an empty `.form-bool` immediately followed by the checkbox. The plugin fills the mount.

## Labels and size

| Field / data attribute | Effect |
| ---------------------- | ------ |
| `text_on` / `data-text-on` | Label inside the track when on. Default `ON`. |
| `text_off` / `data-text-off` | Label when off. Default `OFF`. |
| `data-toggle-width` / `data-toggle-height` | Optional size overrides (px if unitless). |

Default `ON` / `OFF` (any case) is treated as unlabeled: a compact pill with no text. Any other pair is shown inside the track.

Readonly / disabled checkboxes (or a parent `.field.readonly`) render a non-interactive switch.

## Programmatic toggle

Apps that used to drive jquery-toggles can keep the same chain. The plugin stores itself on the mount as both `instance` and `toggles`:

```javascript
$checkbox
    .closest('.field')
    .find('.form-bool')
    .data('toggles')
    .toggle(true);          // on
    // .toggle(false, true, true);  // off, no animation, silent
```

`toggle(state, noanimate, silent)` — omit `state` to flip. Changing the checkbox fires `change` on the input (and a `toggle` event on the mount), so [Revealer](revealer.md) keeps working.
