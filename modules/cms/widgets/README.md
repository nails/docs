---
description: Build, place, and retire CMS widgets, including those supplied by other modules.
---

# Widgets

Widgets are the blocks of content an editor drops into a [widget area](../areas.md), a [page template](../pages/templates.md), or any admin field that uses the widget editor. A widget is a class plus two views: one for admin, one for the front end.

## Anatomy

App widgets live under `application/modules/cms/widgets/{slug}/`. Installed modules use the same layout under `{module}/cms/widgets/{slug}/`.

```
application/modules/cms/widgets/MyWidget/
    widget.php
    screenshot.png
    views/
        editor.php
        render.php
    js/
        dropped.js
        removed.js
```

`widget.php` is the definition. The directory name is the slug (`MyWidget`). The class name is that slug with the first letter uppercased (`MyWidget`), in the `App\Cms\Widget` namespace. A module widget uses the module namespace plus `Cms\Widget`, so the CMS module's accordion widget is `Nails\Cms\Cms\Widget\Accordion` in `cms/widgets/accordion/`.

`screenshot.png` (also `.jpg` or `.gif`) is optional. The editor shows it when the pointer is over the widget in the sidebar. Aim for about 500px wide, with as little surrounding page as possible.

`views/editor.php` is the admin form. `views/render.php` is the front-end markup. Either file can be omitted; a missing view renders as an empty string.

`js/dropped.js` runs when an instance is added to the editor. `js/removed.js` runs after the editor confirms removal. A `.min.js` file is preferred when both are present. Both are optional. Each file is turned into a function that receives the instance's DOM element as `domElement`. `this` is the editor.

## Creating a widget

```bash
nails make:cms:widget MyWidget
```

The command writes the directory, `widget.php`, both views, and `js/dropped.js` under `application/modules/cms/widgets/`.

### Definition

```php
// application/modules/cms/widgets/MyWidget/widget.php

namespace App\Cms\Widget;

use Nails\Cms\Widget\WidgetBase;

class MyWidget extends WidgetBase
{
    public function __construct()
    {
        parent::__construct();

        $this->label       = 'My Widget';
        $this->icon        = 'fa-cube';
        $this->grouping    = 'Generic';
        $this->description = 'A short description about the widget';
        $this->keywords    = 'some,searchable,keywords';

        // Keys defined here are available as variables in both views
        $this->data = [
            'sBody' => '<p>Default body text</p>',
        ];
    }
}
```

| Property | Role |
| --- | --- |
| `$label` | Name in the sidebar and on each placed instance |
| `$icon` | Font Awesome class. Empty falls back to `DEFAULT_ICON` (`fa-cube`) |
| `$grouping` | Sidebar group. An empty value, or the label `Generic`, joins the Generic group |
| `$description` | Shown on the placed instance |
| `$keywords` | Extra search terms for the sidebar filter |
| `$data` | Default values. Missing keys are filled before either view loads |
| `$assets_editor` | Styles and scripts for the editor |
| `$assets_render` | Styles and scripts for the front end |

Asset entries are a URL or path string, or a `[path, location]` pair passed to the asset service.

The editor lists groups with a numeric order first, then unordered groups. Ties sort by label. To give a group an order, overload the widget service with `App\Cms\Service\Widget` (see [Overloading](../../../key-concepts/factory/overloading.md)) and override `getWidgetGroupOrder()` so it returns an integer for that label.

### Editor view

The editor reads every named field in `views/editor.php` and stores it under that name. Checkboxes are booleans, `name[]` fields are arrays, and the payload is JSON so types survive the round trip. Prefer the [form field helpers](../../../key-concepts/form-fields.md) so the widget picks up the same [admin chrome](../../admin/forms.md) as the rest of admin.

```php
// application/modules/cms/widgets/MyWidget/views/editor.php

echo form_field_textarea([
    'key'     => 'sBody',
    'label'   => 'Body',
    'default' => $sBody,
    'tip'     => 'Shown on the front end as the widget body.',
]);
```

### Render view

`views/render.php` receives the saved fields as variables. Rendering fires `WIDGET:RENDER:PRE` and `WIDGET:RENDER:POST` (`Nails\Cms\Events`). Listeners receive the widget, its data, and the output so far; the output argument is a reference.

```php
// application/modules/cms/widgets/MyWidget/views/render.php

if (!empty($sBody)) {
    ?>
    <div class="cms-widget">
        <?=$sBody?>
    </div>
    <?php
}
```

A saved area is a list of instances:

```json
[
    {"slug": "MyWidget", "data": {"sBody": "<p>Hello</p>"}}
]
```

### Javascript

```javascript
// application/modules/cms/widgets/MyWidget/js/dropped.js

domElement
    .querySelector('.some-class')
    .addEventListener('click', function () {
        // ...
    });
```

## Constants

Declare these on the widget class. Each one changes whether the widget can be chosen, edited, or rendered.

| Constant | Default | Effect |
| --- | --- | --- |
| `DISABLED` | `false` | The widget is never instantiated |
| `HIDDEN` | `false` | Kept out of the editor sidebar; existing placements still render |
| `DEPRECATED` | `false` | Kept out of the sidebar; existing placements stay editable and show a warning |
| `ALTERNATIVE` | `''` | Replacement named in that warning |
| `DEFAULT_ICON` | `'fa-cube'` | Icon used when `$this->icon` is empty |

### Disabled

`DISABLED = true` drops the widget during discovery. The sidebar, `getBySlug()`, and `cmsWidget()` all miss it. An area that still references the slug skips it in production. Outside production, rendering that area throws `Nails\Cms\Exception\Widget\NotFoundException`.

Use this when a module ships a widget the app should not offer at all. The usual way is an app widget with the same slug that extends the module class and sets the constant.

### Hidden

`HIDDEN = true` leaves the widget out of the editor sidebar (`Widget::getAvailable()`). `getBySlug()` loads hidden widgets, so a placement that is already saved still renders, and `cmsWidget()` can still target the slug.

The editor's catalogue is that same sidebar list. Opening an area which already contains a hidden widget shows the instance as missing, because the editor cannot look the slug up. Choose `DEPRECATED` when editors still need to open and adjust existing placements.

### Deprecated

`DEPRECATED = true` retires a widget without removing it. The sidebar skips it, so it cannot be added again. A placement that is already saved still opens, and the editor shows "This widget is deprecated." Set `ALTERNATIVE` to the replacement's name or slug and the warning adds "Consider using {alternative} instead." The [monitor](../monitor.md) flags the same state on the widget's detail screen.

```php
namespace App\Cms\Widget;

use Nails\Cms\Widget\WidgetBase;

class MyWidget extends WidgetBase
{
    const DEPRECATED  = true;
    const ALTERNATIVE = 'MyOtherWidget';

    public function __construct()
    {
        parent::__construct();

        $this->label = 'My Widget';
    }
}
```

## Supplied widgets

The CMS module ships these widgets. Slugs match the directory names.

| Slug | Label | |
| --- | --- | --- |
| `richtext` | Rich Text | CKEditor field `body`, wrapped in `.cms-widget-richtext`. See [Rich Text](rich-text.md). |
| `html` | Plain Text | Unfiltered HTML in `body`, wrapped in `.cms-widget-html`. |
| `blockquote` | Blockquote | `quote`, `cite_text`, and `cite_url`. |
| `table` | Table | Handsontable editor. `tblData` is the cell JSON; `tblAttr` is extra markup on the `<table>`. |
| `tabs` | Tabs | Repeatable `title` and `body` fields, rendered as Bootstrap tabs. `cmsWidget()` can pass a `tabs` list of `title` / `body` pairs instead. |
| `accordion` | Accordion | Repeatable `title` and `body` fields, rendered as a Bootstrap collapse group. `cmsWidget()` can pass a `panels` list of `title`, `body`, and `collapsed`. |
| `Area` | CMS Area | Renders another [area](../areas.md) chosen by `iAreaId`. |

Other modules add widgets of their own. Those widgets are documented with the module:

* [CDN](../../cdn/README.md#cms-widget) supplies `image`
* [Custom Forms](../../other/custom-forms.md#cms-widget) supplies `customform`

## Rendering one widget

`cmsWidget()` renders a single widget by slug. The helper is autoloaded with the rest of the CMS helpers. Hidden widgets resolve; disabled widgets return an empty string.

```php
<?=cmsWidget('MyWidget', ['sBody' => '<p>This is some body text.</p>'])?>
```

## Overriding a module widget

Discovery walks installed modules, then the app. An app directory with the same slug replaces the module widget. Extend the module class when the views and behaviour should stay, and only the constants or copy need to change. `getFilePath()` walks the parent classes, so an override can keep the parent's `views/` and `js/` files.

```php
// application/modules/cms/widgets/accordion/widget.php

namespace App\Cms\Widget;

class Accordion extends \Nails\Cms\Cms\Widget\Accordion
{
    const DISABLED = true;
}
```

The directory name has to match the module widget's slug exactly (`accordion`, including case). The class is `Accordion` because the loader uppercases the first letter.

The same pattern with `HIDDEN` or `DEPRECATED` applies when the widget should remain in existing content.
