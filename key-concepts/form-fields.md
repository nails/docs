---
description: Common form_field helpers used in admin and elsewhere.
---

# Form fields

Nails renders labelled admin fields through helpers in `nails/common` (`form_field()`, `form_field_boolean()`, `form_field_dropdown()`, …). CDN and CMS add a few more (`form_field_cdn_object_picker()`, `form_field_cms_widgets()`). The markup is the same shape so [admin chrome](../modules/admin/forms.md) can style it.

```php
echo form_field([
    'key'        => 'label',
    'label'      => 'Label',
    'default'    => $oItem->label ?? '',
    'required'   => true,
    'tip'        => 'Shown in lists and search.',
    'max_length' => 150,
]);
```

## Field config

| Key            | Description |
| -------------- | ----------- |
| `key`          | Input name (and usually the column). |
| `label`        | Visible label. |
| `default`      | Value; `set_value()` wins after a failed POST. |
| `required`     | Visual required marker. Not HTML5 `required`. |
| `tip`          | Short plain string (or `['title' => …, 'class' => …]`). Rendered as a question-mark beside the label. |
| `info`         | Longer copy under the control. May contain HTML. |
| `info_class`   | Extra class on the info element (e.g. `alert alert-info`). |
| `sub_label`    | Small line under the label. |
| `max_length`   | Live [character count](../modules/admin/javascript/character-count.md) on text-like fields. |
| `class`        | Class on the control (`select2` for [Select](../modules/admin/javascript/select.md)). |
| `data`         | `data-*` attributes. `revealer` + `reveal-on` together are copied onto the field container so the whole row can show/hide. |
| `readonly`     | Disables the control; admin shows a padlock on it. |
| `error`        | Force the error state / message. |
| `placeholder`  | Placeholder text. |
| `autocomplete` | Defaults to on. |

The second argument to `form_field($aField, $sTip)` is a deprecated alias of `$aField['tip']`.

### Tip versus info

Use `tip` for a short sentence the user might need while filling the field. Use `info` (optionally with `info_class`) for HTML, alerts, URLs, and operational copy that should stay on screen.

### Required marker

`Nails\Common\Helper\Form\Field::requiredMarker(true)` returns the asterisk plus a visually hidden “required”. Colour lives in admin `.field-required`. Helpers call this for you when `required` is true.

Do not add HTML5 `required` on admin edit fields if the form must be savable as a draft.

## Booleans

```php
echo form_field_boolean([
    'key'      => 'is_published',
    'label'    => 'Status',
    'text_on'  => 'Published',
    'text_off' => 'Draft',
    'default'  => false,
]);
```

Admin paints these as [switches](../modules/admin/javascript/toggles.md). The posted value is still the checkbox.

## Revealer

To hide a whole field when another control changes, put both keys in `data`:

```php
echo form_field_dropdown([
    'key'     => 'template',
    'label'   => 'Login template',
    'options' => $aTemplates,
    'data'    => [
        'revealer'  => 'password,user-details',
        'reveal-on' => 'true',
    ],
]);
```

A `<select>` cannot be a Revealer *element* (it can be a *control*). Wrapping via `form_field` is the usual fix — the helper puts `data-revealer` / `data-reveal-on` on the `.field` container. See [Revealer](../modules/admin/javascript/revealer.md).

## Helpers

From `nails/common` (loaded in admin):

`form_field`, `form_field_text`, `form_field_textarea`, `form_field_email`, `form_field_password`, `form_field_number`, `form_field_url`, `form_field_tel`, `form_field_color`, `form_field_date`, `form_field_time`, `form_field_datetime`, `form_field_dropdown`, `form_field_dropdown_multiple`, `form_field_boolean`, `form_field_radio`, `form_field_checkbox`, `form_field_wysiwyg`, `form_field_wysiwyg_basic`, `form_field_html`, `form_field_json`, `form_field_upload`, `form_field_timecode`.

From other modules:

* [CDN object picker](../modules/cdn/admin.md#cdn-object-pickers)
* [CMS widgets](../modules/cms/helpers.md)
* [Admin dynamic table](../modules/admin/javascript/dynamic-table.md) (`form_field_dynamic_table()`)
