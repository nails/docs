# Resources

Resources are generic re-usable, structured objects which have no real relation to anything, but provide the developer with a consistent structure. New resources can be created by the [Factory](./) and are populated on demand.&#x20;

## Where do resources live?

Resources live at `src/Resource` and in the `App/Resource` namespace. All resources must extend `Nails\Common\Resource`.

## How are resources loaded?

Resources should be loaded using the `Factory` and are defined within the `resources` element of `services.php`. Each time you load a resource you are given a new instance.

Define resources in `application/services/services.php`:

```php
use App\Resource;

return
    'resources' => [
        'Author' => function ($mObject): Resouce\Author {
            return new Resource\Author($mObject);
        },
        'Book' => function ($mObject): Resouce\Book {
            return new Resource\Book(mObject);
        },
        'BookReview' => function ($mObject): Resouce\Book\Review {
            return new Resource\Book\Review($mObject);
        },
    ],
];
```

{% hint style="info" %}
Remember to add the `$mObject` parameter. This will contain the raw data to populate the resource.
{% endhint %}

To use a resource, you load it using the [Factory](./):

```php
use App\Resource;
use Nails\Factory;

/** @var Resource\Book $oBook */
$oBook = Factory::service(
    'Book',
    'app',
    [
        'title' => 'Treasure Island',
    ]
);
```

## Entities

Entity Resources are distributed by models.
