---
description: Forms that can be embedded in a page, including a CMS widget.
---

# Custom Forms

The Custom Forms module stores forms that editors manage under Custom Forms in admin. A form can be placed in a CMS widget area with the widget this module supplies.

## CMS widget

Slug `customform`, grouped under **Custom Forms** (class `Nails\CustomForms\Cms\Widget\Customform`). How widgets are discovered, overridden, and retired is covered in [CMS widgets](../cms/widgets/).

The editor lists existing forms and three booleans:

| Field | Role |
| --- | --- |
| `formId` | The form to render |
| `showLabel` | Print the form label as a heading |
| `showHeader` | Render the form's header widget data above the fields |
| `showFooter` | Render the form's footer widget data below the fields |

Header and footer are themselves widget data, rendered with `cmsAreaWithData()`. The fields are rendered with `formBuilderRender()`. When no forms exist, the editor shows a warning instead of the dropdown.

`cmsWidget()` can pass a form resource as `form` in place of `formId`.

```php
<?=cmsWidget('customform', [
    'formId'     => 3,
    'showLabel'  => true,
    'showHeader' => true,
    'showFooter' => false,
])?>
```