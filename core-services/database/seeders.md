---
description: This page covers database seeders, automatic seeding, and seed helpers.
---

# Seeders

Database seeding is an extremely useful part of the development process; seeding allows you to generate content for your database tables automatically.

Seeders exist under the `App\Seed` namespace and should implement the `Nails\Common\Console\Traits\Database\Seed` interface.&#x20;

## Creating Seeders

It's simplest to generate seeders on the CLI using the `make:db:seed <class>` command –  multiple classes can be passed in at once:

```bash
nails make:db:seed Book,Book\\Review,Movie
# Generates: App\Database\Seed\Book
# Generates: App\Database\Seed\Book\Review
# Generates: App\Database\Seed\Movie
```

{% hint style="warning" %}
Note the double slash in the above command, this is because backslash is an escape character so for the literal slash to be parsed by the command, it needs to be escaped.
{% endhint %}

The above will generate three empty seeders, ready to be customised:

* `App\Database\Seed\Book`
* `App\Database\Seed\Book\Review`
* `App\Database\Seed\Movie`

Which will each look something like this:

```php
namespace App\Database\Seed;

use Nails\Common\Interface\Database\Seed;

class Book implements Seed
{
    public function execute(): void
    {
        // @todo - seed logic
    }
}
```

From here, you are free to write your own seeding logic, [defining distinct entities](seeders.md#defined-entities), or looping a number of times and generating random values using the [seed helpers](seeders.md#seed-helpers).

## The Default Seeder

The `DefaultSeeder` is a utility class which removes a lot of the boilerplate surrounding seeding a model. It binds to a model and will automatically populate the model's table with data which is the correct type

### Creation

`DefaultSeeder` seeds should also be generated on the CLI using the `make:db:seed:model <model>` command. Multiple models can be passed in at once:

```bash
nails make:db:seed:model Book,BookReview,Movie
# Generates: App\Database\Seed\Book
# Generates: App\Database\Seed\BookReview
# Generates: App\Database\Seed\Movie
```

{% hint style="warning" %}
Note that `BookReview` is not namespaced as `Book\Review` this is because the literal string for loading the model via the Factory is `BookReview`. You are free to change the directory structure of seeders after they have been created.
{% endhint %}

### Configuration

The seeder, at minimum should provide the following constant: `CONFIG_MODEL_NAME`. This is the model name, as required by the [Factory](../../key-concepts/factory/).

```php
namespace App\Database\Seed;

use Nails\Common\Database\Seed\DefaultSeed;

class Book extends DefaultSeed
{
    const CONFIG_MODEL_NAME = 'Book';
}
```

The following example shows all the configurable constants, and their default values

```php
namespace App\Database\Seed;

use Nails\Common\Console\Database\Seed\DefaultSeed;

class Book extends DefaultSeed
{
    /**
     * The model to bind this seeder to
     */
    const CONFIG_MODEL_NAME     = '';
    const CONFIG_MODEL_PROVIDER = 'app';
    
    /**
     * The number of items to create
     */
    const CONFIG_NUM_PER_SEED = 20;
    
    /**
     * Fields to explicitly ignore when generating
     */
    const CONFIG_IGNORE_FIELDS = [
        'id',
        'is_deleted',
        'created',
        'created_by',
        'modified',
        'modified_by',
    ];
}
```

### Data Generation

The `DefaultSeeder` analyses the model and determines the fields which should be populated, and their data type. It delegates the actual data generation to the `generate(array $aFields)` method which returns key/value pairs of appropriate data. Often, however, the data (whilst correct from a data type point of view) is incompatible with the logic of the application, for example: an integer field may be a foreign key.

In these situations you may wish to override the `generate(array $aFields)` method and provide your own logic, or you may simply wish to tweak the returned values. For example, in a hypothetical `Book` seeder, you may wish to specify an Author ID, and require that this be set.

```php
protected function generate($aFields): array
{
    $aData = parent::generate($aFields);
    
    /** @var Model\Author $oAuthorModel */
    $oAuthorModel = Factory::model('Author', 'app');
    
    $aData['foreign_key'] = $oAuthorModel->getRandom()->id ?? null;
    if (empty($aData['foreign_key'])) {
        throw new \Exception('All books must have an author.');
    }
    
    return $aData;
}
```

{% hint style="info" %}
You will probably find [Seed Helpers](seeders.md#seed-helpers) most useful here.
{% endhint %}

## Seed Helpers

> @todo - write this up

## Running Seeders

To execute seeders, you should use the `db:seed` command:

```bash
nails db:seed
```

If you wish to seed an empty database, use the `--fresh` flag:

```bash
nails db:seed --fresh
```

If you just want to see what seeders are available, use the `--list` flag:

```bash
nails db:seed --list

The following seeders are available::

- [app] Book
- [app] Book\Review
- [app] Movie
- [vendor/module] Some\Seeder
```

### Filtering

The `db:seed` command allows you filter the seeders by component and class name:

#### Show seeders provided by the app

```bash
nails db:seed app --list

The following seeders are available::

- [app] Book
- [app] Book\Review
```

#### Show specific seeders provided by the app

```bash
nails db:seed app Book,Book\\Review --list

The following seeders are available::

- [app] Book
- [app] Book\Review
- [app] Movie
```

{% hint style="warning" %}
Note the double slash in the above command, this is because backslash is an escape character so for the literal slash to be parsed by the command, it needs to be escaped.
{% endhint %}

## Defined Entities

> @todo - write this up
