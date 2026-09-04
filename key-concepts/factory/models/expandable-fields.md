---
description: Nails manages inter-model relationships using ExpandableFields.
---

# Expandable Fields

Often items returned by models reference other items (or many other items). The following `Book` object, for example, references the author in the `author_id` field in a 1-to-1 relationship:

```javascript
{
    "id": 1,
    "label": "Treasure Island",
    "author_id": 23
}
```

It would be desirable to expand this ID into the actual object, so we'd have an object which looks like this:

```javascript
{
    "id": 1,
    "title": "Treasure Island",
    "author": {
        "id": 23,
        "label": "Robert Louis Stephenson"
    }
}
```

In addition, there might be various items which reference back to the book via its ID. For example, this book might have many reader reviews (in a 1-to-many relationship), which we'd like to embed in the main object:

```javascript
{
    "id": 1,
    "title": "Treasure Island",
    "author": {
        "id": 23,
        "label": "Robert Louis Stephenson"
    },
    "reviews": {
        "count": 2,
        "data": [
            {
                "id": 123,
                "comment": "Great read!",
                "number_stars": 4,
                "user_id": 12
            },
            {
                "id": 356,
                "comment": "Not bad, not bad at all.",
                "number_stars": 3,
                "user_id": 24
            }
        ]
    }
}
```

To achieve this, we need to define some expandable fields on the `Book` model, and then specify what we want expanded when querying the `Book` model.

We will use the example above in the descriptions below, with a table structure like this:

```bash
# Book
id int(11) unsigned
title varchar(150)
author_id int(11) unsigned

# Author
id int(11) unsigned
label varchar(150)

# BookReview
id int(11) unsigned
book_id int(11) unsigned
comment text
number_stars int(11) unsigned
user_id int(11) unsigned
```

## Defining Relationships

### One-to-One

One-to-one relationships are where a [resource](../resources.md) has a property/column which contains the ID of a [resource](../resources.md) provided by another [model](./).&#x20;

Define these using the `hasOne($sTrigger, $sModel)` method when constructing the model.

Using the `Book` example above, we would define its relationship with the `Author` model like so:

```php
public function __construct()
{
    $this->hasOne('author', 'Author');
}
```

This method assumes that model is provided by `app` and that the ID column is `$sTrigger` appended with `_id` – you can change this by manually specifying a third and a fourth argument.

{% hint style="info" %}
It is convention to use singular trigger words for one-to-one relationships
{% endhint %}

### One-to-Many

One-to-many relations are when a related [resource](../resources.md) has a property/column which contains the ID of the current [resource](../resources.md).&#x20;

Define these using the `hasMany($sTrigger, $sModel, $sForeignColumn)` method when constructing the model.

Using the `Book` example above, we would define its relationship with the `BookReview` model like so:

```php
public function __construct()
{
    $this->hasMany('reviews', 'BookReview', 'book_id');
}
```

This method assumes that the model is provided by `app` you can change this by manually specifying a fourth argument.

{% hint style="info" %}
It is convention to use plural trigger words for one-to-many relationships.
{% endhint %}

### Many-to-Many

Expandable field definitions are _one way_, meaning that a relationship declared on one model does not imply the reverse/inverse on the related model.

You can, of course, specify the inverse relationship manually. For example, on the `Author` model you might define the following relationship:

```php
public function __construct()
{
    $this->hasMany('books', 'Book', 'author_id');
}
```

Triggering this expansion would return all `Book` resources whose `author_id` is set to the current `Author` resource.

### Manual Definitions

The above utility methods are syntactic sugar for the `addExpandableField(array $aOptions)` method provided by `\Nails\Common\Model\Base`.

The `$aOptions` parameter accepts an array with the following indexes:

| Key         | Description                                                                                                                                                                                                                                                                               |            Default           | Required |
| ----------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :--------------------------: | :------: |
| `trigger`   | The keyword which will trigger the expansion, and which will be the property the expanded object is attached to.                                                                                                                                                                          |           `<none>`           |    Yes   |
| `type`      | <p>The type of expansion: single or many</p><p>This must be one of: <code>static::EXPAND_TYPE_SINGLE</code> or <code>static::EXPAND_TYPE_MANY</code></p>                                                                                                                                  | `static::EXPAND_TYPE_SINGLE` |    No    |
| `model`     | The name of the model which will get the related object, as defined in the [services file](../#defining-factory-items).                                                                                                                                                                   |           `<none>`           |    Yes   |
| `provider`  | The provider of the `<model>`.                                                                                                                                                                                                                                                            |             `app`            |    No    |
| `id_column` | <p>The ID column to use.</p><p>For <code>EXPAND_TYPE_SINGLE</code> this is the property of the parent object which contains the related object's ID.</p><p>For <code>EXPAND_TYPE_MANY</code> this is the property on the <em>related</em> object which contains the parent item's ID.</p> |           `<none>`           |    Yes   |
| `data`      | A control array to pass to the related model.                                                                                                                                                                                                                                             |             `[]`             |          |

{% hint style="warning" %}
Note that definitions are one way, and apply _only_ to the model on which it is defined. You are free to add an expandable field in other child models which reference "back" to the parent.
{% endhint %}

## Triggering Expansions

In order to trigger an expandable fields you need to pass in some data when querying a model using any of its `get*()` methods, specifically you need to tell it which items you would like expanded.

{% hint style="success" %}
Nails provides helpers to do this for you, skip to the [Helpers](expandable-fields.md#helpers) section if want an easy life.
{% endhint %}

It is possible to "expand" items by passing in an `expand` element to the configuration array when calling one of the `get*()`, methods e.g.:

{% tabs %}
{% tab title="PHP" %}
```php
use App\Model;
use App\Resource;
use Nails\Factory;

/** @var Model\Book $oModel */
$oModel = Factory::model('Book', 'app');

/** @var Resource\Book[] $aResults */
$aBooks = $oModel->getAll([
    'expand' => [
        'author',
        'reviews'
    ]
]);
```
{% endtab %}

{% tab title="Output" %}
```javascript
[
    {
        "id": 1,
        "title": "Treasure Island",
        "author": {
            "id": 23,
            "name": "Robert Louis Stevenson"
        },
        "reviews": {
            "count": 2,
            "data": [
                {
                    "id": 123,
                    "comment": "Great read!",
                    "number_stars": 4
                },
                {
                    "id": 356,
                    "comment": "Not bad, not bad at all.",
                    "number_stars": 3
                }
            ]
        }
    }
]
```
{% endtab %}
{% endtabs %}

It is also possible to pass a configuration array to the expanded field's model. To do this, instead of using a string in the `expand` array, pass instead an array where the first element is the trigger and the second element is the configuration array you want to pass to the associated item's model.

For example:

```php
use App\Model;
use App\Resource;
use Nails\Factory;

/** @var Model\Book $oModel **/
$oModel = Factory::model('Book');

/** @var Resource\Book[] $aBooks */
$aBooks = $oModel->getAll([
    'expand' => [
    
        //  This is a basic expansion, i.e the trigger word as a string
        'author',
        
        //  This is equivalent to the above
        [
            'author',
            []
        ],
        
        //  This is a nested expansion, where we're passing in an array
        //  to the Book\Review Model
        [
            //  The trigger word
            'reviews',
            //  To pass to the expanded model's getAll() method
            [
                'where' => [
                    ['rating >', 3]
                ],
                'sort' => [
                    ['created', 'desc']
                ],
                
                // You can pass in nested expansions too!
                'expand' => [...]
            ]
        ]
    ]
]);
```

In the above example, the expansions are happening like this:

```php
Model\Book->getAll()
    ↳ Model\Author->getById($oBook->id)
    ↳ Model\Book\Review->getAll([
        'where' => [
            ['book_id', $oBook->id]
        ]
    ])
```

## Saving expandable fields

Multiple/many expandable fields can be used when writing as well as reading. To save expandable fields you must pass an array of arrays using the trigger keyword as the element.

For example, to save a book's reviews whilst also updating its title you could do this. First, let's imagine the existing data looks like this:

```javascript
{
    "id": 123,
    "title": "Island of Treasure",
    "reviews": {
        "count": 2,
        "data": [
            {
                "id": 123,
                "comment": "Great read!",
                "number_stars": 2,
                "user_id": 12
            },
            {
                "id": 456,
                "comment": "A terrible book",
                "number_stars": 0,
                "user_id": 25
            }
        ]
    }
}
```

Now, if we were to want to do the following:

* Correct the book's title to be `Treasure Island`
* Update the review with ID `123` to be `4` stars instead of `2`
* Add a new review which reads `Loved it!`
* Delete the review with ID `456`

We can do this in a single operation, which look like this:

```php
use App\Model;
use App\Resource;
use Nails\Factory;

/** @var Model\Book $oModel **/
$oModel = Factory::model('Book');
$oModel->update(
    123,
    [
        'title'   => 'Treasure Island',
        'reviews' => [
            [
                'id'           => 123,
                'number_stars' => 4,
            ],
            [
                'comment'      => 'Loved it!',
                'number_stars' => 5,
                'user_id'      => 38
            ]
        ]
    ]
);
```

In the above example we are passing two reviews, as well as setting the book's label, and author ID. There are four important things to note here:

#### The `review` key is not a column, but the expandable field's trigger

`title` is a column on the `app_books` table, whilst `review` is the trigger as defined by the model's `hasMany()` method;

#### The `id` column will trigger an update instead of an insert

Notice the first review has an `id` key - this is the ID of the existing review in the example `app_book_review` table. If you know the ID then passing it will cause the model to _update_ this record rather than _insert_ a new one (notice the second review has no `id`).

{% hint style="info" %}
In this example we do not need to pass all the fields as we're only updating a single field.
{% endhint %}

#### Existing items not included are deleted

We wanted to delete review with ID `456` so we simply don't include it in the update operation. The IDs of all items are tracked and anything not updated or created is discarded.

#### We do not pass the book\_id column to the expandable fields

We do not need to pass this column as the model will set it automatically – it already knows that this column is the foreign key as it is defined in the model's `hasMany` method.

## Helpers

Expanding fields can get complex and confusing quickly, especially when nesting multiple items. The syntax can trip people up as it is particularly wordy. For this reason, Nails provides helper classes to do this for you in an object-orientated way.

### Single Expands

The `Nails\Common\Helper\Model\Expand` class accepts two constructor parameters:

1. The trigger keyword for the expansion you wish to execute
2. An optional configuration for the expansion. If supplied, this can be:
   1. A control array to pass into the expanded model
   2. Another instance of `Nails\Common\Helper\Model\Expand`
   3. An instance of `Nails\Common\Helper\Model\Expand\Group`

The code example below shows various combinations:

```php
use App\Model\Book;
use Nails\Common\Helper\Model\Expand;
use Nails\Factory;

/** @var Book $oModel */
$oModel = Factory::model('Book', 'app');
$oModel-getAll([
    'expand' => [
        //    A simple expand
        new Expand('author'),
        
        //    Expand, and apply a control array
        new Expand(
            'reviews',
            [
                'where' => [
                    ['rating >', 3]
                ]
            ]
        ),
        
        //    A nested expand
        new Expand(
            'reviews',
            new Expand('user'),
        ),
    ]
]);
```

### Grouping Expands

Expands can be grouped easily using the `Nails\Common\Helper\Model\Expand` helper class.

```php
use App\Model\Book;
use Nails\Common\Helper\Model\Expand;
use Nails\Factory;

/** @var Book $oModel */
$oModel = Factory::model('Book', 'app');
$oModel-getAll([
    'expand' => new Expand\Group(
        new Expand('author'),
        new Expand('reviews'),
    )
]);
```

{% hint style="info" %}
You can omit the `expand` array property when using the helpers; the root of the control array will be searched for instances of the helpers and automatically expanded for you.
{% endhint %}
