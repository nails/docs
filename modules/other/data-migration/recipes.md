---
description: >-
  Recipes deifne how a Unit of data is transformed once it leaves the source
  Connector, and before it is sent to the target Connector.
---

# Recipes

A recipe defines which properties of the source should be exposed to the target, and if any mutation should happen. Recipies are used by Pipelines.

Recipes implement the `\HelloPablo\DataMigration\Interfaces\Recipe` trait.

{% hint style="info" %}
Typically a Recipe and a Pipeline will have a one-to-one relationship, but it's possible for the same recipe to be used by many pipelines if the data it represents is similar.
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
         * Each transformation for a unit of work must be yielded
         */
    }
}
```
