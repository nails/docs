---
description: Front-end helpers for blocks, menus, pages, areas, and widgets.
---

# Helpers

The CMS module autoloads two helpers: `cms` and `form`. Call them directly. To load them yourself:

```php
use Nails\Cms\Constants;
use Nails\Factory;

Factory::helper('cms', Constants::MODULE_SLUG);
Factory::helper('form', Constants::MODULE_SLUG);
```

## Admin

### form_field_cms_widgets()

Renders the button that opens the widget editor. The model field type is `Nails\Cms\Helper\Form::FIELD_WIDGETS` (`cms_widgets`). It is a [form field](../../key-concepts/form-fields.md), so `tip`, `required`, and `info` work the same as other helpers.

```php
echo form_field_cms_widgets([
    'key'   => 'body',
    'label' => 'Body',
    'tip'   => 'Widgets are rendered in order on the front end.',
]);
```

The posted value is a JSON list of `{slug, data}` objects. See [Widgets](widgets/).

## Front end

### cmsBlock($mIdSlug)

Returns the block's stored value for an id or slug, or an empty string when the block is missing.

Image and file blocks store a CDN object id. `cmsBlock()` returns that id. Call `render()` on the block resource when you want the served URL, or use a short tag inside a template. See [Blocks](blocks.md#on-the-front-end).

### cmsMenu($mIdSlug, $aData = [])

Returns a `Nails\Cms\Resource\Menu`, or `null`. `$aData` is passed through to the menu model. See [Menus](menus.md).

### cmsPage($mIdSlug)

Returns a `Nails\Cms\Resource\Page`, or `null`. The resource has `published` and `draft` data. See [Pages](pages/).

### cmsArea($mIdSlug)

Returns the rendered HTML for an area id or slug. An unknown area returns an empty string. See [Areas](areas.md).

### cmsAreaWithData($mWidgetData)

Renders a widget list you already hold (array or JSON) and returns HTML.

### cmsWidget($sSlug, $aData = [])

Renders one widget by slug. Hidden widgets resolve. A disabled or unknown slug returns an empty string. See [Widgets](widgets/#rendering-one-widget).
