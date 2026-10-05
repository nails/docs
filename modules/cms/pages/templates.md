---
description: Layouts that define widget areas and options for CMS pages.
---

# Templates

A template lays a [page](./) out. It declares widget areas, which editors fill with [widgets](../widgets/), and optional fields that change the layout.

## Anatomy

App templates live at `application/modules/cms/templates/{slug}/`. Modules use `{module}/cms/templates/{slug}/`.

```
application/modules/cms/templates/MyTemplate/
    template.php
    view.php
    icon.png
```

`template.php` is the definition. The directory name is the slug. The class name is that slug with the first letter uppercased, in `App\Cms\Template`. `view.php` is the full HTML for the page, including any header and footer the layout needs. `icon.png` (also `.jpg`, `.jpeg`, or `.gif`) is shown in the page editor.

Generate a stub with:

```bash
nails make:cms:template MyTemplate
```

```php
namespace App\Cms\Template;

use Nails\Cms\Constants;
use Nails\Cms\Template\TemplateBase;
use Nails\Factory;

class MyTemplate extends TemplateBase
{
    public function __construct()
    {
        parent::__construct();

        $this->label       = 'My Template';
        $this->description = 'A short description about the template';

        $this->widget_areas = [
            'mainbody' => Factory::factory('TemplateArea', Constants::MODULE_SLUG)
                ->setTitle('Main Body')
                ->setDescription('Primary page content'),
            'sidebar' => Factory::factory('TemplateArea', Constants::MODULE_SLUG)
                ->setTitle('Sidebar'),
        ];
    }
}
```

The widget area's array key is the variable in `view.php`. The example above exposes `$mainbody` and `$sidebar`, each a string of rendered widget HTML.

```php
<?=$mainbody?>
<aside><?=$sidebar?></aside>
```

Templates can also set `$assets_editor` and `$assets_render`, the same shape as [widget assets](../widgets/#definition).

Set `protected static $isDefault = true` to sort the template first within its group. The bundled full-width template does this.

## Template options

`$additional_fields` is a list of `TemplateOption` objects. Each option is a [form field](../../../key-concepts/form-fields.md). The view variable is the option's key (`setKey()`), and the value is whatever the editor saved.

```php
$this->additional_fields = [
    Factory::factory('TemplateOption', Constants::MODULE_SLUG)
        ->setType('dropdown')
        ->setKey('number_of_columns')
        ->setLabel('No. of columns')
        ->setDefault(2)
        ->setOptions([
            '2' => '2 Columns',
            '3' => '3 Columns',
            '4' => '4 Columns',
        ]),
];
```

`view.php` then reads `$number_of_columns`.

After the view renders, block short tags in the HTML are replaced. See [Blocks](../blocks.md#on-the-front-end).

## Constants

| Constant | Default | Effect |
| --- | --- | --- |
| `DISABLED` | `false` | The template is left out of discovery |
| `DEPRECATED` | `false` | Flagged in the [monitor](../monitor.md) and on the template payload (`is_deprecated`) |
| `ALTERNATIVE` | `''` | Replacement named beside that flag |

## Bundled templates

| Slug | Label | |
| --- | --- | --- |
| `fullwidth` | Full Width | One `mainbody` area. Marked as the default template. |
| `sidebar` | Sidebar | `mainbody` and `sidebar`. Options `sidebarWidth` (1–6 columns) and `sidebarSide` (`LEFT` or `RIGHT`). |
| `columns` | Columns | `col1`–`col4`. Options `numColumns` (2–4) and `breakpoint` (`xs`, `sm`, `md`, `lg`). |
| `redirect` | Redirect | Sends the request to another page (`redirect_page_id`) or URL (`redirect_url`). `redirect_code` is `302` or `301`. A URL wins over a page. An empty target is a 404. |

## Overriding a module template

The app is scanned last. A template directory with the same slug replaces the module template. Extend the module class to keep its areas and view; `getFilePath()` walks parent classes, so the parent's `view.php` is used when the app template does not include one.

The bundled full-width class is `Nails\Cms\Cms\Template\Fullwidth`. The app directory must be `fullwidth` so the slug matches.

```php
namespace App\Cms\Template;

class Fullwidth extends \Nails\Cms\Cms\Template\Fullwidth
{
    public function __construct()
    {
        parent::__construct();

        $this->label       = 'My Full Width Template';
        $this->description = 'I have overridden the default Full Width template.';
    }
}
```
