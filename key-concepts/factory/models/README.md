# Models

In Nails, models represent a database table; all interactions with that table should be done by querying the model.

## Where do models live?

Models live at `src/Model` and in the `App\Model` namespace. All models must extend `Nails\Common\Model\Base`.

## How are models loaded?

Models should be loaded using the `Factory` and are defined within the `models` element of `services.php`. Each time you load a model you are given the same instance.

Define models in `application/services/services.php`:

```php
use App\Model;

return
    'models' => [
        'Author' => function (): Model\Author {
            return new Model\Author();
        },
        'Book' => function (): Model\Book {
            return new Model\Book();
        },
        'Book\Review' => function (): Model\Book\Review {
            return new Model\Book\Review();
        },
    ],
];
```

To use a model, you load it using the [Factory](../):

```php
use App\Model;
use Nails\Factory;

/** @var Model\Author $oAuthorModel */
$oAuthorModel = Factory::model('Author', 'app');

/** @var Model\Book $oBookModel */
$oBookModel = Factory::model('Book', 'app');

/** @var Model\Book\Review $oBookModel */
$oBookReviewModel = Factory::model('BookReview', 'app');
```

## The Base model

Models must extend `Nails\Common\Model\Base`; the base model provides a consistent and predictable CRUDy API for all models, as well as enables the use of [Expandable Fields](./#expandable-fields).

At minimum, a model must define the table it binds to, as well as the [resource](../resources.md) it dispenses; for example:

```php
namespace App\Model;

use Nails\Common\Model\Base;

class Book extends Base {
    const TABLE             = APP_DB_PREFIX . 'book';
    const RESOURCE_NAME     = 'Book';
    const RESOURCE_PROVIDER = 'app';
}
```

{% hint style="info" %}
Many more configuration options are also available and are defined as constants in the base model.
{% endhint %}

### Querying records

The `getAll` method is the primary way of querying the model; it accepts a configuration array which allows you to reduce the result set, as well as supply pagination parameters. By default, a query to `getAll()` with no parameters will return the entire contents of the database as an array.

```php
use App\Model;
use App\Resource;
use Nails\Factory;

/** @var Model\Book $oModel */
$oModel = Factory::model('Book', 'app');

/** @var Resource\Book[] $aBooks */
$aBooks = $oModel->getAll();
```

{% hint style="info" %}
Also available are `getById()`and `getBySlug()`for retrieving a record by its ID, or slug.
{% endhint %}

Typically, unless there is a need for the _entire_ table to be returned, this will be called with various `where` conditionals defined to restrict the result set; also typical is to sort results.

```php
use Nails\Common\Helper\Model;

[
    // Apply various where conditions
    new Model\Where($sColumn, $mValue, $bEscape),
    new Model\WhereIn($sColumn, [$mValue]),
    new Model\WhereNotIn($sColumn, [$mValue]),
    new Model\Like($sColumn, $mValue, $bEscape),
    new Model\NotLike($sColumn, $mValue, $bEscape),
    new Model\Having($sColumn, $mValue, $bEscape),

    new Model\OrWhere($sColumn, $mValue, $bEscape),
    new Model\OrWhereIn($sColumn, [$mValue]),
    new Model\OrWhereNotIn($sColumn, [$mValue]),
    new Model\OrLike($sColumn, $mValue, $bEscape),
    new Model\OrNotLike($sColumn, $mValue, $bEscape),
    new Model\OrHaving($sColumn, $mValue, $bEscape),
    
    // For complex/custom conditionals, use Filter
    // to apply a literal conditional
    new Model\Filter($sQuery),

    // Sort
    new Model\Sort($sColumn1, Sort::DESC),
    new Model\Sort($sColumn2, Sort::ASC),
    
    // Pagination
    new Model\Paginate($iPerPage, $iPage)
]
```

{% hint style="info" %}
All the blocks will be contained within parenthesis and joined with `AND`. i.e `(where conditions) AND (or_where conditions)`.
{% endhint %}

An an example query getting the first 5 books by a particular author, which are published, sorted by the date they were published:

```php
use App\Model;
use App\Resource;
use Nails\Common\Helper\Model;
use Nails\Factory;

/** @var Model\Book $oModel */
$oModel = Factory::model('Book', 'app');

/** @var Resource\Book[] */
$aBooks = $oModel->getAll([

    new Model\Where('author_id', 123),
    new Model\Where('is_published', true),
    
    new Model\Sort('publish_date'),
    
    new Model\Paginate(5),
]);

```

### Creating records

Records can be created by passing in a key/value array to the model's `create()` method.

```php
use App\Model;
use Nails\Factory;

/** @var Model\Book $oModel */
$oModel = Factory::model('Book', 'app');

/** @var int $iId */
$iId = $oModel->create([
    'label'  => 'Treasure Island',
    'author' => 'Robert Louis Stevenson'
]);
```

The `create()` method also accepts a boolean as the second parameter, setting this to `true` will trigger the method to return the newly created object, rather than the object's ID.

```php
use App\Model;
use App\Resource;
use Nails\Factory;

/** @var Model\Book $oModel */
$oModel = Factory::model('Book', 'app');

/** @var Resource\Book $oBook */
$oBook = $oModel->create(
    [
        'label'  => 'Treasure Island',
        'author' => 'Robert Louis Stevenson'
    ],
    true
);
```

{% hint style="info" %}
Should an error occur, the method will return `null`, the reason for failure can be retrieved using the model's `lastError()` method.
{% endhint %}

### Updating records

Records can be updated by passing in the item's ID and a key/value array to the model's `update()` method.

```php
use App\Model;
use Nails\Factory;

/** @var Model\Book $oModel */
$oModel  = Factory::model('Book', 'app');

/** @var bool $bResult */
$bResult = $oModel->update(
    123,
    [
        'label' => 'Treasure Island - the return of Black Beard',
    ]
);
```

### Deleting records

Records can be deleted by passing in the item's ID model's `delete()` method.

```php
use App\Model;
use Nails\Factory;

/** @var Model\Book $oModel */
$oModel  = Factory::model('Book', 'app');

/** @var bool $bResult */
$bResult = $oModel->delete(123);
```

{% hint style="info" %}
Models can be configured to be non-destructive. If a model is non-destructive then to permanently delete a record you must use the `destroy()` method.
{% endhint %}

## Relationships

Often items returned by models reference other items (or many other items). The following `Book` object, for example, references the author in the `author_id` field in a 1-to-1 relationship:

```javascript
{
    "id": 1,
    "title": "Treasure Island",
    "author_id": 23
}
```

In addition, there might be various items which reference back to the book via its ID. For example, this book might have many reader reviews (in a 1-to-many relationship).

Nails has a powerful relationship system called [ExpandableFields](expandable-fields.md), which is big enough to warrant its own section.

{% content-ref url="expandable-fields.md" %}
[expandable-fields.md](expandable-fields.md)
{% endcontent-ref %}

## Describing Fields

The base model exposes the `describeFields()` method for describing a model's fields. For the most part this is entirely automatic and simply translates a `DESCRIBE table;` query into an array of `Nails\Common\Factory\Model\Field` objects. `Field` objects can be used by modules (e.g. [Admin](../../../modules/admin/)) to, for example, generate UI  or validate user input.

The `describeFields` method will do its best to guess the data type and validation rules based on the data available to it. For example, if a field is a `varchar(150)` it will apply a `MAX_LENGTH[150]` validation rule.

There will be times, however when you need to override the behaviour, or add additional information (like validation rules, classes, info, or fieldsets). This can all be done by overloading the `describeFields` method in your model and setting properties as required.

```php
namespace App\Model;

use Nails\Common\Model\Base;
use Nails\Common\Factory\Model\Field;

class Book extends Base
{
    // ... configs ommitted for brevity

    /**
     * return Field[]
     */
    public function describeFields()
    {
        // This is an array of Field objects, configured using a
        // best guess of the available data
        $aFields = parent::describeFields();

        // Adjust the `author_*` fields to read more easily
        // and to be in their own fieldset
        $aFields['author_name']
            ->setLabel('Author: Name')
            ->setFieldset('Author');

        $aFields['author_url']
            ->setLabel('Author: URL')
            ->setType(\Nails\Common\Helper\Form::FIELD_URL)
            ->setFieldset('Author');

        // Change the field type for the `body` field to be a CMS widget editor
        // Note we're using the helper provided by the CMS module            
        $aFields['body']
            ->setType(\Nails\Cms\Helper\Form::FIELD_WIDGETS)
        
        return $aFields;
    }
}
```

The following `Field` properties are available to edit:

| Property       | Description                                                                                                         |
| -------------- | ------------------------------------------------------------------------------------------------------------------- |
| `key`          | The fields key. This is usually the column name in the database, but might also be the name of an expandable field. |
| `label`        | The visible label for the field.                                                                                    |
| `type`         | The field's type, should be one of the `\Nails\Common\Helper\Form::FIELD_*` constants.                              |
| `validation[]` | Validation rules. This will be pre-populated using known data, but you may wish to add more rules.                  |
| `default`      | The default value for the field.                                                                                    |
| `options[]`    | Key/value options for `select` type fields.                                                                         |
| `max_length`   | The maximum length of the field.                                                                                    |
| `class`        | A class to apply to the field (used in [Admin](../../../modules/admin/)).                                           |
| `info`         | Any extra info, hints, or tips to show to to show to the user (used in [Admin](../../../modules/admin/)).           |
| `fieldset`     | Which fieldset/tab to group the field in (used in [Admin](../../../modules/admin/)).                                |
| `data`         | A key/value array of `data-` attributes to set (used in [Admin](../../../modules/admin/)).                          |

## Traits

Nails ships with some useful traits for altering/adding to model behaviours:

### Sortable

> @todo - write this up

### Nestable

> @todo - write this up

### Publishable

> @todo - write this up

### Copyable

> @todo - write this up

### Localised

> @todo - write this up
