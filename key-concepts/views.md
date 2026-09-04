# Views

## Views

Views are loaded using the [View service](../core-services/view.md). This service provides a `load()` method:

```php
\Nails\Factory::service('View')
    ->load([
        'structure/header',
        'blog/index',
        'structure/footer',
    ]);
```

This will load the views and append them to the output which is sent to the browser at the end of execution.

{% hint style="info" %}
To immediately return the specified views as a string pass `true` as the third parameter
{% endhint %}

### View Variables

The View service provides a `setData()` method which accepts an array of key/value pairs. Data passed here will be made available to views by accessing the `key` as a variable. For example, the following data:

```php
\Nails\Factory::service('View')
    ->setData([
        'book'   => [
            'label'  => 'Treasure Island',
            'author' => 'Robert Louis Stephenson',
        ],
        'rating' => 5
    ])
    ->load([
        'structure/header',
        'blog/index',
        'structure/footer',
    ]);
```

Would be accessible in the view as `$book['label']` and `$rating`.
