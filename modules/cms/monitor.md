---
description: Keep tabs on where CMS widgets and templates are being used.
---

# Monitor

As sites develop, iterate, and age, it's likely that certain widgets will become obsolete. In a big project it can be difficult to know what widget is being used where. To solve this the CMS module provides a utility tool for monitoring where widgets and templates are used throughout the project.

As widget data is simply serialised into a JSON object, we cannot leverage thigns like foreign keys to let us know _where_ widgets are being used, instead we need to declare in which columns they _can_ be used and then look in those columns for instances of the widget's slug.

We do this by providing mappers. Mappers are classes in the `App\Cms\Monitor` namespace which implement one of the following interfaces:

* `Nails\Cms\Interfaces\Monitor\Widget`
* `Nails\Cms\Interfaces\Monitor\Template`

A monitor's responsibility is to report its label (used when viewing details about a widget or template's usage) as well as a count of how many times the widget is used in that particular context, as well as return an array of usages (also used when viewing details about a widget or template).

For example, if a `Book` model has column `body` which contains widget data; a widget monitor might look like this:

```php
namespace App\Cms\Monitor\Widget

use Nails\Common\Service\Database;
use Nails\Cms\Interfaces;
use Nails\Cms\Factory\Monitor\Detail;

class Book implements Interfaces\Widget
{
    /**
     * Returns the mapper's label, used on the details page
     */
    public function getLabel(): string
    {
        return 'Books';
    }

    // --------------------------------------------------------------------------

    /**
     * Counts the number of instances a given widget is used
     */
    public function countUsages(Interfaces\Widget $oWidget): int
    {
        /** @var Database $oDb */
        $oDb    = Factory::service('Database');
        $oModel = Factory::model('Book', 'app');
        
        $oDb->from($oModel->getTableName());
        
        $oDb->where(
            'JSON_CONTAINS(JSON_EXTRACT(`body`, "$[*].slug"), \'"%s"\', "$")'
        );
        
        return $oDb->count_all_results();
    }

    // --------------------------------------------------------------------------

    /**
     * Locates instances where a given widget is used
     */
    public function getUsages(Interfaces\Widget $oWidget): array
    {
        /** @var Database $oDb */
        $oDb    = Factory::service('Database');
        $oModel = Factory::model('Book', 'app');
        
        $oDb->from($oModel->getTableName());
        
        $oDb->where(
            'JSON_CONTAINS(JSON_EXTRACT(`body`, "$[*].slug"), \'"%s"\', "$")'
        );
        
        return array_map(function (\stdClass $oRow) {

            /** @var Detail\Usage $oUsage */
            $oUsage = Factory::factory(
                'MonitorDetailUsage',
                Constants::MODULE_SLUG,
                
                // The item's label
                $oRow->label,
                
                // The item's "view" URL
                siteUrl('books/' . $oRow->slug),
                
                // The item's "edit" url
                siteUrl('admin/app/book/edit/' . $oRow->id)
            );

        }, $oDb->get()->result());
    }
}
```

## Monitor Trait

To make things a little easier, and to reduce duplication in logic, a trait is provided for both widget and template monitors. These traits both behave in the same way: they provide a structure for defining a table to inspect and specifically which columns to look in for the widget or template's slug.

* `Nails\Cms\Traits\Monitor\Widget`
* `Nails\Cms\Traits\Monitor\Template`

Using the same `Book` example above, the same mapper might be re-written like so:

```php
namespace App\Cms\Monitor\Widget;

use Nails\Cms\Constants;
use Nails\Cms\Interfaces;
use Nails\Cms\Traits;
use Nails\Cms\Factory\Monitor\Detail;
use Nails\Factory;

class Book implements Interfaces\Monitor\Widget
{
    use Traits\Monitor\Widget;

    // --------------------------------------------------------------------------

    /**
     * Returns the mapper's label, used on the details page
     */
    public function getLabel(): string
    {
        return 'Books';
    }

    // --------------------------------------------------------------------------

    /**
     * Returns the table to inspect
     */
    protected function getTableName(): string
    {
        return Factory::model('Book', 'app')->getTableName();
    }

    // --------------------------------------------------------------------------

    /**
     * Returns the columns which contain widget data
     */
    private function getDataColumns(): array
    {
        return ['body'];
    }

    // --------------------------------------------------------------------------

    /**
     * Returns the columsn to use in the detail query, passed as $oRow
     * to compileUsage()
     */
    private function getQueryColumns(): array
    {
        return ['id', 'label', 'slug'];
    }

    // --------------------------------------------------------------------------

    /**
     * Returns a usage object detailing the item which uses the widget
     */
    protected function compileUsage(\stdClass $oRow): Detail\Usage
    {
        /** @var Detail\Usage $oUsage */
        $oUsage = Factory::factory(
            'MonitorDetailUsage',
            Constants::MODULE_SLUG,
            
            // The item's label
            $oRow->label,
            
            // The item's "view" URL
            siteUrl('books/' . $oRow->slug),
            
            // The item's "edit" url
            siteUrl('admin/app/book/edit/' . $oRow->id)
        );

        return $oUsage;
    }
}
```
