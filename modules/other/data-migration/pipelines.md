---
description: >-
  Pipelines represent a single migration, from a source Connector, to a target
  Connector. Pipelines define the source and destination Connectors, as well as
  the Recipe to use when transferring data.
---

# Pipelines

A Pipeline describes a migration. It defines what is being migrated, and what Recipe will be applied to the Unit of work as it flows from the source to thge target. It also allows the developer to hook into various pointsthroughout the lifecycle of a migration.

Pipelines implement the `\HelloPablo\DataMigration\Interfaces\Pipeline` trait.

{% hint style="info" %}
Pipelines should exist in the `App\DataMigration\Pipeline` namespace, and will be automatically discovered.
{% endhint %}

## Connectors

Each Pipeline has a source and a target, these are represented using Connectors and both must be specified using the Pipelines `getSourceConnector` and `getTargetConnector` methods.

{% hint style="info" %}
A Pipeline is only capable of reading from one source, and writing to one source. In a MySQL-to-MySQL migration this would be a source table, and a target table. Additional tables, such as pivot tables, would be represented by a second Pipeline, and leverage [ID Tracking](id-tracking.md).
{% endhint %}

```php
namespace App\DataMigration\Pipeline;

use HelloPablo\DataMigration\Interfaces\Connector;
use HelloPablo\DataMigration\Interfaces\Pipeline;

class Book implements Pipeline
{
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
    
    // Further methods ommited for brevity
}
```

## Recipe

The recipe defines how a unit of data from the source is represented in the target. A recipe defines which properties of the source should be exposed to the target, and if any mutation should happen.

```php
namespace App\DataMigration\Pipeline;

use HelloPablo\DataMigration\Interfaces\Recipe;
use HelloPablo\DataMigration\Interfaces\Pipeline;

class Book implements Pipeline
{
    public function getRecipe(): Recipe
    {
        return new \App\DataMigration\Recipe\Book();
    }
    
    // Further methods ommited for brevity
}
```

## Priority

The pipeline's priorioty is a numeric value which is used when sorting multiple Pipelines into the order in which they should be execute. If a Pipeline is to be executed before or after another pipeline then this numeric value should represent that.

```php
namespace App\DataMigration\Pipeline;

use HelloPablo\DataMigration\Interfaces\Pipeline;

class Book implements Pipeline
{
    public static function getPriority(): int
    {
        return 0;
    }
    
    // Further methods ommited for brevity
}
```

This is a static method so priority dependencies can be linked between pipelines, for example if `Book` must be implemented after the `Author` pipeline, then you might configure it like this:

```php
namespace App\DataMigration\Pipeline;

use HelloPablo\DataMigration\Interfaces\Pipeline;

class Book implements Pipeline
{
    public static function getPriority(): int
    {
        return \App\DataMigration\Pipeline\Author::getPriority() + 1;
    }
    
    // Further methods ommited for brevity
}
```

## Hooks

There are many hooks which are executed throughout a Pipeline's lifecycle. It is recommended that you use the [DefaultBehaviour Trait](pipelines.md#defaultbehaviour-trait), to avoid having to define all the hooks required by the interface.

#### `commitStart()`

Called at the start of the Pipeline's commit step.

#### `commitBefore(Unit $oUnit)`

Called before each unit of work is committed, passed the current unit of work.

#### `commitAfter(Unit $oUnit)`

Called after each unit of work is committed, passed the current unit of work.

#### `commitSkipped(Unit $oUnit, SkipException $e)`

Called if a `SkipException` exception is caught, passed the unit of work and the exception.

#### `commitError(Unit $oUnit, Exception $e)`

Called if an Exception is caught, passed the unit of work and the exception.

#### `commitFinish(array $aErrors)`

Called at the end of the Pipeline's commit step, passed an array of any encountered errors.

## DefaultBehaviour Trait

The `HelloPablo\DataMigration\Traits\Pipeline\DefaultBehaviour` trait defines a priority of `0` and defines all the hooks (which do nothing). This trait is useful to keep Pipeline boiler plate code to a minimum.
