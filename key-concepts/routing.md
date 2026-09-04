# Routing

Routing in Nails is relatively simple. There are three tiers to routing:

* [Explicit Routes](routing.md#explicit-routes)
* [Generated Routes](routing.md#generated-routes)
* [Automatic Routes](routing.md#automatic-routes)

Specificity is important, a match near the top of the chain will always take precedence over something further down.

## Explicit Routes

Explicit application routes are set in the `application/config/routes.php` file and take the following format:

```php
$route['some/url/structure']   = 'module/controller/methodOne';
$route['some/other/structure'] = 'module/controller/methodTwo';
```

When Nails detects a matching URL structure, it will direct the request to the specified controller and execute the specified method. There's no reason multiple routes shouldn't point to the same controller/method.

Routes also support wildcards, allowing you to make dynamic URLs which are all handled by a single controller. You may use regular expressions in routes:

```php
// A single book page, e.g. /book/treasure-island
$route['book/(.+)'] = 'books/single/index';

// Page 5 of the books index, e.g. /books/5
$route['books/(\d+)'] = 'books/books/index';
```

## Generated Routes

Nails also provides the ability for routes to be generated dynamically; this is useful when components (or indeed the application) need to write specific routes dependent on the contents of the database. These are generated on demand when a module determines that the routes need to be updated.

For example, your desired URL structure might be:

```
/book-title
/book-title/reviews
/book-title/reviews/review-id
```

However, `book-title` is a dynamic entity managed by the users, it would be inpractical to have to manually update the routes file each time a book is added.

Here you might want to automatically create routes as `books` are added:

```
'treasure-island'              => 'books/single/index',
'treasure-island/reviews'      => 'books/single/reviews',
'treasure-island/reviews/(.+)' => 'books/single/reviews',
```

To achieve this your app should make available `App\Routes`, a class which should implement the `RouteGenerator` interface. This class should contain a static `generate` method which returns an array of key/value route pairs. Using the book example:

```php
namespace App;

use App\Model;
use Nails\Common\Interfaces\RouteGenerator;
use Nails\Factory;

class Routes implements RouteGenerator
{
    public static function generate()
    {
        $aRoutes = [];
    
        /** @var Model/Book $oModel **/
        $oModel = Factory::model('Book', 'app');

        /**
         * Using the raw query is more efficient when
         * dealing with potentially large result sets
         **/
        $oQuery = $oModel->getAllRawQuery();
        
        while ($oBook = $oQuery->unbuffered_row()) {
            $aRoutes[$oBook->slug]                   = 'book/single/index'
            $aRoutes[$oBook->slug . '/reviews']      = 'book/single/reviews'
            $aRoutes[$oBook->slug . '/reviews/(.*)'] = 'book/single/reviews'
        }
    
        return $aRoutes;
    }
}
```

{% hint style="info" %}
Trigger routes to be rewritten by emitting the `Nails\Common\Events::ROUTES_UPDATE` [event](../core-services/event.md#triggering-events).
{% endhint %}

## Automatic Routes

If no matching explicit or generated route is found then the router switches into automatic mode and attempts to infer the desired controller. It assumes that the URL segments map to controllers in the following way:

```
example.com/<module>/<controller>/<method>
```

| Segment        | Default    |
| -------------- | ---------- |
| `<module>`     | No default |
| `<controller>` | `<module>` |
| `<method>`     | `index`    |

For example, all the following URLs would resolve to the same place:

| URL                | Resolves to       |
| ------------------ | ----------------- |
| /books/books/index | books/books/index |
| /books/books       | books/books/index |
| /books             | books/books/index |

Some further examples to illustrate automatic routing:

| URL                           | Resolves to               |
| ----------------------------- | ------------------------- |
| books/books/index             | books/books/index         |
| books/books/author            | books/books/author        |
| books/books/best-sellers      | books/books/best\_sellers |
| books/reviews                 | books/reviews/index       |
| books/reviews/filter          | books/reviews/filter      |
| books/reviews/filter/positive | books/reviews/filter      |

If the `<module>` segment is not found in `application/modules` then the router will look through the installed components (which can themselves register a particular URL namespace).

{% hint style="info" %}
Note that dashes are automatically translated into underscores; useful as dashes are not valid characters to use in PHP Class and method names
{% endhint %}

## Default Route / Home page

By default, the homepage resolves to `home/home/index`. If you wish to change this, then you may define the following magic route, `application/config/routes.php`:

```php
$route['default_controller'] = 'your/custom/route';
```
