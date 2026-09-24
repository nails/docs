---
description: Small, reusable snippets of content managed from admin.
---

# Blocks

A block is a single named snippet of content (plaintext, rich text, image, file, email, number, or URL). The front end loads it by slug; see [Helpers](helpers.md).

## Admin

Create and edit blocks through CMS → Blocks. The create screen is a [DefaultController](../admin/controllers/default-controller.md) form (label, slug, type, …). Once the block exists, the edit screen shows those details as a read-only table and the value through the matching [form field](../../../key-concepts/form-fields.md):

| Type | Helper |
| ---- | ------ |
| Plaintext | `form_field_textarea()` |
| Rich text | `form_field_wysiwyg()` |
| Image / file | `form_field_cdn_object_picker()` |
| Email | `form_field_email()` |
| Number | `form_field_number()` |
| URL | `form_field_url()` |

Save uses the admin [floating save bar](../admin/forms.md#floating-save-bar).
