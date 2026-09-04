---
description: >-
  This page defines a simple migration of two similar, but not identical,
  tables.
---

# Example

The following demonstrates how a simple table-to-table migration would work between two tables which are similar, but not identical. In this example we're migrating a `books` table from a legacy database into a new database. The tables are largely the same, but the data requires some mutating before it is isnerted into the new table.

This migration does **not** demonstrate ID tracking, prioritising Pipelines, or Pipeline hooks.

## Table Structure

Consider the following tables:

### Source Table

This is our legacy table, we intend to migrate the data from this table in our old database, to an equivalent table in the new database.

```sql
CREATE TABLE `books` (
  `id` int unsigned NOT NULL AUTO_INCREMENT,
  `title` varchar(150) DEFAULT NULL,
  `description` varchar(150) DEFAULT NULL,
  `author` varchar(150) DEFAULT NULL,
  `date_published` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

### Target Table

This is the target table in our new database. It is functionally very similar, but the data will not copy across without some mutation:

* `label` is the title field for the book
* `excerpt` is the description field
* addition of required column `is_published`
* `date_published` is a `date` column, not a timestamp

```sql
CREATE TABLE `books` (
  `id` int unsigned NOT NULL AUTO_INCREMENT,
  `label` varchar(150) DEFAULT NULL,
  `excerpt` varchar(150) DEFAULT NULL,
  `author` varchar(150) DEFAULT NULL,
  `is_published` tinyint(1),
  `date_published` date DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

## Classes

The following classes will need to be defined for this migration:

### Pipeline

The defintion for this migration will look like this:

```php
namespace App\DataMigration\Pipeline;

use App\DataMigration;
use HelloPablo\DataMigration\Interfaces\Connector;
use HelloPablo\DataMigration\Interfaces\Pipeline;
use HelloPablo\DataMigration\Interfaces\Recipe;
use HelloPablo\DataMigration\Traits;
use Nails\Common\Exception\FactoryException;
use Nails\Common\Exception\ModelException;
use Nails\Factory;

class Blog implements Pipeline
{
    // Using this trait sets default behaviours and reduces boilerplate
    use Traits\Pipeline\DefaultBehaviour;

    // Configure a connector to read from the source table
    public function getSourceConnector(): Connector
    {
        return new \HelloPablo\DataMigration\Connector\MySQL(
            new \HelloPablo\DataMigration\Unit(),
            'db_source_host',
            'db_source_user',
            'bd_source_password',
            3306,
            'db_source_name',
            'books'
        );
    }

    // Configure a connector to write to the target table
    public function getTargetConnector(): Connector
    {
        return new \HelloPablo\DataMigration\Connector\MySQL(
            new \HelloPablo\DataMigration\Unit(),
            'db_target_host',
            'db_target_user',
            'bd_target_password',
            3306,
            'db_target_name',
            'books'
        );
    }

    // Define the recipe to use for this Pipeline
    public function getRecipe(): Recipe
    {
        return new DataMigration\Recipe\Book();
    }
}
```

### Recipe

The recipe for this Pipeline looks like this:

{% hint style="info" %}
Note that we must `yield` a transformer for each column we wish to insert into the target table, with the exception of the `id` column, which is tracked automatically.
{% endhint %}

```php
namespace App\DataMigration\Recipe;

use App\DataMigration;
use App\DataMigration\Transformer\TimestampToDate;
use HelloPablo\DataMigration\Exception\DataMigrationException;
use HelloPablo\DataMigration\Interfaces\Recipe;
use HelloPablo\DataMigration\Transformer\Copy;
use HelloPablo\DataMigration\Transformer\Set;

class Book implements Recipe
{
    public function yieldTransformers(): \Generator
    {
        /**
         * The Copy transformer doesn't perform any mutations but
         exposes the source colum `title` as `label`.
         */
        yield new Copy('title', 'label');
        yield new Copy('description', 'excerpt');
        
        /**
         * Even if the columns are identical, both must be specified
         */
        yield new Copy('author', 'author');
        
        /**
         * `is_published` must be set on the target table, in this
         * instance we are assuming that all items are publsihed
         * and we simply need to `Set` the value
         */
        yield new Set(null, 'is_published', true);
        
        /**
         * `date_published` must be transformed from a timestamp (int)
         * to a MySQL date field, we'll do this using our custom
         * `TimeStampToDate` Transformer
         */
        yield new TimestampToDate('date_published', 'date_published');
    }
}
```

### Transformers

We are primarily using bundled Transformers, but our Recipe requires us to provide some custom behaviour, i.e. convert a Unix timestamp into a date string.

The following Transfromer achieves this for us:

```php
namespace App\DataMigration\Transformer;

use HelloPablo\DataMigration\Interfaces\Unit;
use HelloPablo\DataMigration\Transformer\Copy;

class TimestampToDate extends Copy
{
    public function transform($mInput, Unit $oUnit)
    {
        try {
        
            return \DateTime::createFromFormat('U', $mInput)
                ->format('Y-m-d');
            
        } catch (\Exception $e) {
            return null;
        }
    }
}
```

## Running the migration

To execute the migration we use the `datamigration:run` console command:

```
nails datamigration:run
```

This will prepare the migration, and then commit it. use the `--help` flag to learn more about what this command can do, e.g. dry runs, filtering which pipelines to run, and debug options.
