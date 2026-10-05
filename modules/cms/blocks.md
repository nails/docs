---
description: Small, reusable snippets of content managed from admin.
---

# Blocks

A block is a single named snippet of content (plaintext, rich text, image, file, email, number, or URL). The front end loads it by id or slug; see [On the front end](#on-the-front-end).

## Admin

Create and edit blocks through CMS → Blocks. The create screen is a [DefaultController](../admin/controllers/default-controller.md) form (label, slug, type, …). Once the block exists, the edit screen shows those details as a read-only table and the value through the matching [form field](../../key-concepts/form-fields.md):

| Type | Helper |
| ---- | ------ |
| Plaintext | `form_field_textarea()` |
| Rich text | `form_field_wysiwyg()` |
| Image / file | `form_field_cdn_object_picker()` |
| Email | `form_field_email()` |
| Number | `form_field_number()` |
| URL | `form_field_url()` |

Save uses the admin [floating save bar](../admin/forms.md#floating-save-bar). The type is fixed once the block exists.

## On the front end

`cmsBlock($mIdSlug)` returns the stored value. For plaintext, rich text, email, number, and URL that is the text the editor entered. For an image or file it is the CDN object id.

The block resource's `render()` method returns that same text, and for an image or file returns the URL from `cdnServe()`.

CMS templates also replace short tags in the rendered HTML. A tag is the block slug wrapped as `[:slug:]`. Replacement uses `render()`, so an image short tag becomes the served URL.

```php
<?=cmsBlock('phone-number')?>
```

```php
use Nails\Cms\Constants;
use Nails\Factory;

$oBlock = Factory::model('Block', Constants::MODULE_SLUG)->getByIdOrSlug('hero');

if ($oBlock) {
    echo $oBlock->render();
}
```
