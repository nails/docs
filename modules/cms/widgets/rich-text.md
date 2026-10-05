---
description: The bundled rich text widget.
---

# Rich Text

Slug `richtext`. The editor is a CKEditor 4 textarea named `body`. On the front end the saved HTML is printed inside `.cms-widget.cms-widget-richtext`.

The widget loads CKEditor from its editor asset list, so the field is ready inside any widget editor, including [page](../pages/) widget areas. A [block](../blocks.md) of type rich text uses `form_field_wysiwyg()` on its own.

```php
<?=cmsWidget('richtext', ['body' => '<p>Hello</p>'])?>
```

For unfiltered markup, use the Plain Text widget (`html`). It stores the same `body` field and prints it inside `.cms-widget-html`.
