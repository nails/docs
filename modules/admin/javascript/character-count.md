---
description: Live character counters on admin text fields.
---

# Character Count

When a text field has a maximum length, admin sits a live `current / max` caption in a channel along the bottom of the control rather than as an extra row between fields.

`form_field()` emits this automatically when you pass `max_length` for text, textarea, email, password, number, or URL fields:

```php
echo form_field([
    'key'        => 'excerpt',
    'label'      => 'Excerpt',
    'max_length' => 160,
]);
```

The plugin looks for `small.char-count` inside `.field` with `data-max-length`, wraps the `.field-input`, and updates the caption as the user types. Exceeding the limit adds `.max-length-exceeded` on the field.

The counter is visual. It does not set HTML `maxlength` and does not block submit; enforce the limit with validation (`max_length[160]` on the model, or your own rule).
