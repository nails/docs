---
description: Transformers mutate a single piece of data within a Recipe.
---

# Transformers

Transformers are used by Recipes to mutate data between the source and target connectors. A transformation might be a simple as copying a value, or completely custom behaviour using callbacks.

Transformers implement the `\HelloPablo\DataMigration\Interfaces\Transformer` trait.

## Bundled Transformers

The following transformers are bundled with the module:

### Copy

This transformer performs no mutations, simply copying the value from `source_col` to `target_col` exatly as it appears:

```php
yield new \HelloPablo\DataMigration\Transformer\Copy(
    'source_col',
    'target_col'
);
```

### Set

No source column is expected, returns the value as specified as the third argument:

```php
yield new \HelloPablo\DataMigration\Transformer\Set(
    null,
    'target_col',
    'value'
);
```

### Callback

Passes the source value to a callback function which returns the new value.

```php
yield new \HelloPablo\DataMigration\Transformer\Callback(
    'source_col',
    'target_col',
    function($mInput, $oUnit) {
        /**
         * Mutates $mInput, can inspect $oUnit if necessary
         */
         return $mInput * 100;
    )
);
```

### Date

Parses a value as a date and returns a formatted date string, pass a default value as the third parameter (used if `col_a` is an invalid value).

```php
yield new \HelloPablo\DataMigration\Transformer\Date(
    'source_col',
    'target_col',
    new \DateTime()
);
```

{% hint style="info" %}
Defaults to `Y-m-d`, change this using the `setFormat` method.
{% endhint %}

### DateTime

As `Date` but includes a time component.

### Id

Maps an old ID to a new ID, understand more about how this works in [ID Tracking](id-tracking.md).

```php
yield new \HelloPablo\DataMigration\Transformer\Id(
    'source_col',
    'target_col',
    \App\Pipeline\Book::class
);
```

### Merge

Merges multiple source columns into a single value using `glue`.

```php
yield new \HelloPablo\DataMigration\Transformer\Merge(
    'source_col',
    'target_col',
    ['another_source_col']
);
```

The above would result in the values of `source_col` and `another_source_col` being concactenated using a space.

### Slug

Generates a value sutiable for a URL from a `source_col`:

```php
yield new \HelloPablo\DataMigration\Transformer\Slug(
    'source_col',
    'target_col'
);
```

### StripHtml

Removes all HTML from a string:

```php
yield new \HelloPablo\DataMigration\Transformer\StripHtml(
    'source_col',
    'target_col'
);
```

### Trim

Trims whitespace from a string:

```php
yield new \HelloPablo\DataMigration\Transformer\Trim(
    'source_col',
    'target_col'
);
```

### Truncate

Truncates a string to a maximum length:

```php
yield new \HelloPablo\DataMigration\Transformer\Truncate(
    'source_col',
    'target_col'
    // $iMaxLength, maximum length of string, defaults to 150
    // $sEllipsis, string to use for the ellipsis, defaults to ...
);
```

## Grouping Transformers

Each target property must be represented by a single Transformer. In roder to perform a series of transformations on a property you must use the `Group` transformer:

```php
yield new \HelloPablo\DataMigration\Transformer\Group(
    'source_col',
    'target_col',
    [
        new \HelloPablo\DataMigration\Transformer\StripHtml(),
        new \HelloPablo\DataMigration\Transformer\Truncate()
    ]
);
```

{% hint style="info" %}
Note that we supply an array of transfromer instances, and we do not specify any column data outside of the `Group` transformer.
{% endhint %}
