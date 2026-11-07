---
description: Forms that can be embedded in a page, including a CMS widget.
---

# Custom Forms

The Custom Forms module stores forms that editors manage under Custom Forms in admin. A form can be placed in a CMS widget area with the widget this module supplies.

## CMS widget

Slug `customform`, grouped under **Custom Forms**. The v1 class is `Nails\CustomForms\Cms\Widget\Customform`. WidgetEditor (v2) uses `Nails\CustomForms\Cms\Widgets\Customform`, which answers to the same slug and the same stored keys. How widgets are discovered, overridden, and retired is covered in [CMS widgets](../cms/widgets/).

The editor lists existing forms and three switches:

| Field | Role |
| --- | --- |
| `formId` | The form to render. An empty choice until one is picked. |
| `showLabel` | Print the form label as a heading |
| `showHeader` | Render the form's header widget data above the fields |
| `showFooter` | Render the form's footer widget data below the fields |

When no forms exist, the form field says so and tells the editor to create one under Custom Forms. Header and footer are themselves widget data, rendered with `cmsAreaWithData()`. The fields are rendered with `formBuilderRender()`. A missing form renders nothing. On the canvas, no form asks for one. A chosen form uses the same dotted box as an area: the widget's icon, the form's name, then "This form will be rendered here." The label, header and footer are still drawn there when their switches are on.

`cmsWidget()` can pass a form resource as `form` in place of `formId`.

```php
<?=cmsWidget('customform', [
    'formId'     => 3,
    'showLabel'  => true,
    'showHeader' => true,
    'showFooter' => false,
])?>
```