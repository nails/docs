---
description: Report where CMS widgets and templates are used.
---

# Monitor

Widgets and templates end up as JSON (or a slug column) on other tables, so usage is not a foreign key. The monitor asks each mapper where to look, then counts and lists the rows.

Admin shows two screens under Utilities, for users with the matching permission:

* CMS Monitor: Widgets
* CMS Monitor: Templates

The widget screen includes hidden widgets and excludes disabled ones. A deprecated widget or template is flagged on its detail screen, including the `ALTERNATIVE` text when one is set.

## Mappers

A mapper is any class under a component's `Cms\Monitor` namespace that implements one of:

* `Nails\Cms\Interfaces\Monitor\Widget`
* `Nails\Cms\Interfaces\Monitor\Template`

The app namespace is `App\Cms\Monitor`. The CMS module ships mappers for areas (`Nails\Cms\Cms\Monitor\Widget\Area`) and for pages (widget data and template slugs).

Each mapper returns a label, a usage count, and a list of `Nails\Cms\Factory\Monitor\Detail\Usage` objects (label, optional view URL, optional edit URL).

### Widget trait

`Nails\Cms\Traits\Monitor\Widget` runs a `JSON_CONTAINS` query. By default it looks for the widget slug at `$[*].slug`. Override `getJsonPath()` when the JSON is shaped differently. The slug and the path are escaped, so a slug may contain any character.

A mapper that builds its own SQL should not interpolate the slug. `Nails\Cms\Helper\Monitor::jsonString()` returns the slug as a JSON string (as MySQL prints it), for `JSON_CONTAINS`. `likePattern()` returns a `LIKE` pattern for that string, with `%` and `_` escaped, to use with `ESCAPE '!'`. Neither result is SQL-quoted: pass it through the Database service's `escape()`.

```php
namespace App\Cms\Monitor\Widget;

use Nails\Cms\Constants;
use Nails\Cms\Factory\Monitor\Detail;
use Nails\Cms\Interfaces;
use Nails\Cms\Traits;
use Nails\Factory;

class Book implements Interfaces\Monitor\Widget
{
    use Traits\Monitor\Widget;

    public function getLabel(): string
    {
        return 'Books';
    }

    protected function getTableName(): string
    {
        return Factory::model('Book', 'app')->getTableName();
    }

    protected function getDataColumns(): array
    {
        return ['body'];
    }

    protected function getQueryColumns(): array
    {
        return ['id', 'label', 'slug'];
    }

    protected function compileUsage(\stdClass $oRow): Detail\Usage
    {
        return Factory::factory(
            'MonitorDetailUsage',
            Constants::MODULE_SLUG,
            $oRow->label,
            siteUrl('books/' . $oRow->slug),
            siteUrl('admin/app/book/edit/' . $oRow->id)
        );
    }
}
```

### Template trait

`Nails\Cms\Traits\Monitor\Template` has the same methods. It matches the template slug with a plain column comparison, which fits columns such as `published_template` and `draft_template`.

Implement the interface directly when the lookup is not a JSON slug list or a single column.
