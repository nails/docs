---
description: How Admin discovers controllers, routes to them, builds the sidebar, and finds their views.
---

# Controllers

An admin controller is a class that renders one area of admin, such as _Books_ or _Orders_. Each one decides which sidebar links it adds and which permissions its methods need.

## Where controllers live

Admin finds controllers by namespace. It checks every installed component (your app and every module) for instantiable classes in that component's `Admin\Controller` namespace that implement `Nails\Admin\Interfaces\Controller`.

For your app, that means `App\Admin\Controller`, which autoloads from `src/Admin/Controller/`:

```
src/
  Admin/
    Controller/
      Book.php              → App\Admin\Controller\Book
      Shop/
        Product.php         → App\Admin\Controller\Shop\Product
```

A module works the same way from its own namespace. For example, a blog module might provide `Nails\Blog\Admin\Controller\Post`.

In practice you don't implement the interface yourself. You extend one of the two base classes:

* [`Nails\Admin\Controller\Base`](base-controller.md) is the foundation for every admin controller. It loads the admin environment and gives you `url()`, titles, breadcrumbs, and view data helpers. Use it for custom screens.
* [`Nails\Admin\Controller\DefaultController`](default-controller.md) extends `Base` and builds a full CRUD interface for a model. Use it for most data-management screens.

{% hint style="warning" %}
Admin controllers aren't CodeIgniter controllers. Don't put them in `application/controllers` or `application/modules/admin/controllers`, because Admin won't discover them there.
{% endhint %}

## Routing

Admin registers a single route that sends every URL under the admin prefix (`admin` by default, set by `ADMIN_URL`) to its router. The router maps URL segments like this:

```
/admin/{component}/{controller}/{method}/{...args}
```

| Segment        | Comes from                                                                                                                |
| -------------- | ------------------------------------------------------------------------------------------------------------------------- |
| `{component}`  | The URL slug of the component that provides the controller. Your app is `app`.                                           |
| `{controller}` | The class name relative to `Admin\Controller`, dash-cased, with namespace separators turned into dashes.                 |
| `{method}`     | The public method to call. Defaults to `index`.                                                                          |
| `{...args}`    | Ignored by the router. Read them with the `Uri` service, for example an item's ID.                                       |

| Class                               | URL                          |
| ----------------------------------- | ---------------------------- |
| `App\Admin\Controller\Book`         | `/admin/app/book`            |
| `App\Admin\Controller\BookReview`   | `/admin/app/book-review`     |
| `App\Admin\Controller\Shop\Product` | `/admin/app/shop-product`    |

Visiting `/admin` on its own redirects to the dashboard.

If a controller defines a `_remap()` method, Admin calls that for every request to the controller instead of the method named in the URL.

{% hint style="info" %}
Don't hardcode admin URLs. Use the controller's static `url()` method instead: `Book::url()` returns the controller's base URL and `Book::url('edit/' . $iId)` appends to it. It respects `ADMIN_URL`, and it follows [overrides](#overriding-a-modules-controller).
{% endhint %}

## The sidebar: `announce()`

Every controller has a static `announce()` method. It returns a `Nav` object (or an array of them, or `null`) that describes which sidebar group the controller adds links to:

```php
use Nails\Admin\Constants;
use Nails\Admin\Factory\Nav;
use Nails\Factory;

public static function announce(): Nav|array|null
{
    /** @var Nav $oNav */
    $oNav = Factory::factory('Nav', Constants::MODULE_SLUG);
    $oNav
        ->setLabel('Books')
        ->setIcon('fa-book')
        ->addAction('Manage Books')              // → index
        ->addAction('Reviews', 'reviews');       // → reviews()

    return $oNav;
}
```

The action URL is relative to the controller, so `'reviews'` links to `Book::url('reviews')`.

A few rules apply:

* **Groups merge by label.** Several controllers, even from different modules, can add links to the same group (for example `Settings`). If they disagree on the icon, the most common one wins. Add `!important` to the icon to force it.
* **Empty groups are hidden.** Only add actions the current user is allowed to use, and a user with no access won't see the group at all. Returning `null` keeps the controller routable but leaves it out of the sidebar.
* **Order.** _Dashboard_ comes first, then the other groups in alphabetical order, then _Settings_ and _Utilities_. Users can reorder and collapse groups, and Admin remembers their choice.

`addAction()` also accepts alerts (badges such as a count of pending items), an explicit order, and search keywords for the sidebar filter:

```php
$oNav->addAction(
    'Pending Reviews',
    'reviews?status=pending',
    [
        Factory::factory('NavAlert', Constants::MODULE_SLUG)
            ->setValue($iPending)
            ->setSeverity('danger'),
    ],
    null,
    ['moderation', 'queue']
);
```

## Views

Views are PHP templates, so they don't live in `src/`. Load them with `Helper::loadView('name')` (or `$this->loadView('name')`). Admin works out the directory from the controller's class name, relative to `Admin\Controller`, with namespaces becoming subdirectories:

```
application/
  modules/
    admin/
      views/
        Book/
          reviews.php          ← App\Admin\Controller\Book::reviews()
        Shop/
          Product/
            index.php          ← App\Admin\Controller\Shop\Product::index()
```

{% hint style="warning" %}
File casing matters. The view directory must match the controller's class name _exactly_.
{% endhint %}

`loadView()` wraps the view in the admin header and footer. Use `Helper::loadInlineView()` to render a partial with no chrome. See [Helper](../helper/).

A module's controllers look for views in the module's own `admin/views/` directory. Controllers that extend `DefaultController` fall back to Admin's built-in `DefaultController` views, so you only need to supply the views you want to change.

### View data

Anything you put in the controller's `$data` array is available as a variable in the view. `setData()` is a chainable shortcut:

```php
public function reviews(): void
{
    $oModel = Factory::model('Review', 'app');

    $this
        ->setTitles(['Books', 'Reviews'])
        ->setData('aReviews', $oModel->getAll());

    Helper::loadView('reviews');
}
```

```php
<!-- application/modules/admin/views/Book/reviews.php -->
<table class="table table-striped table-hover">
    <tbody>
        <?php foreach ($aReviews as $oReview): ?>
            <tr>
                <td><?=$oReview->body?></td>
            </tr>
        <?php endforeach; ?>
    </tbody>
</table>
```

## Overriding a module's controller

To change how a module's admin screen behaves, extend its controller in your app, in the same relative namespace:

```php
namespace App\Admin\Controller;

class Post extends \Nails\Blog\Admin\Controller\Post
{
    const CONFIG_SIDEBAR_GROUP = 'Content';
}
```

The app is searched first. Any module controller that one of your app's controllers extends is dropped from discovery, so the sidebar shows your version and the module's `url()` calls resolve to it too.

The overriding controller still uses the module's views. To replace one, put a file with the same name in `application/modules/{module}/admin/views/{Controller}/`. Admin checks there before the module's own `admin/views/` directory.

## In this section

{% content-ref url="base-controller.md" %}
[base-controller.md](base-controller.md)
{% endcontent-ref %}

{% content-ref url="default-controller.md" %}
[default-controller.md](default-controller.md)
{% endcontent-ref %}

{% content-ref url="breadcrumbs.md" %}
[breadcrumbs.md](breadcrumbs.md)
{% endcontent-ref %}
