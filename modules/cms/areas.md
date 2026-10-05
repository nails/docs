---
description: Reusable collections of widgets, rendered by slug.
---

# Areas

An area is a named list of [widgets](widgets/). Editors create them under CMS → Areas. Each area has a label, a slug (taken from the label), a description for other editors, and the widget data itself.

The description and label stay in admin. The front end receives the rendered widgets.

```php
<?=cmsArea('homepage-hero')?>
```

`cmsArea()` accepts the area id or slug and returns HTML. An unknown area returns an empty string.

`cmsAreaWithData()` renders a widget list you already have, as an array or a JSON string, without loading an area row. The [Custom Forms](../other/custom-forms.md#cms-widget) widget uses it for a form's header and footer.

```php
<?=cmsAreaWithData($aWidgetData)?>
```

Outside production, a widget slug that cannot be resolved throws `Nails\Cms\Exception\Widget\NotFoundException`. In production that instance is skipped. Disabled widgets are unresolved; hidden widgets still render. See [widget constants](widgets/#constants).

The bundled `Area` widget embeds one area inside another widget area. Its editor is a dropdown of existing areas (`iAreaId`).

Widget data is JSON in the `widget_data` column. The [monitor](monitor.md) ships a mapper for that column (`Nails\Cms\Cms\Monitor\Widget\Area`).
