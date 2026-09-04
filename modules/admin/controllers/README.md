# Controllers

## Loading Controllers

Admin will look for controllers in the app's `application/module/admin/controllers` directory. Classes it finds here must be in the `App\Admin\App` namespace.  Valid classes will be loaded and have their `announce` method called which will return actions available for the current user.

### The `Base` Controller&#x20;

All app-supplied admin controllers must extend the `Nails\Admin\Controller\Base` admin controller. The base controller is responsible for bootstrapping the admin environment.

{% content-ref url="base-controller.md" %}
[base-controller.md](base-controller.md)
{% endcontent-ref %}

### The `DefaultController`

Bundled with the module is the `DefaultController`. This controller is an abstract class which you can extend which binds to a model and  provides advanced out-of-the box functionality for many day-to-day administration needs.

{% content-ref url="default-controller.md" %}
[default-controller.md](default-controller.md)
{% endcontent-ref %}

## Routing

All admin routes live under the `/admin` URL namespace. Controllers supplied by the app (rather than by installed [components](../../../key-concepts/components/)) are accessible under the `/admin/app` namespace.

App admin controllers resolve as follows:

```
/admin/app/{controller}/{method}
```

So, for example, a `Book` controller containing the method `reviews` would be accessible at:

```
/admin/app/book/reviews
```

{% hint style="info" %}
Any additional URL segments are ignored by the router and can be used to pass arguments to the Admin controller, e.g. an object's ID.
{% endhint %}

## Views

Views in admin are loaded via the [Admin Helper](../helper/). This helper will intelligently load up a view for that particular controller.

For example, consider the following directory structure:

```
application/
  modules/
    admin/
      controllers/
        Book.php
      views/
        Book/
          index.php
          reviews.php
```

The `Book` controller might look like this :

```php
namespace App\Admin\App;

use Nails\Admin;
use Nails\Factory;

class Book extends Admin\Controller\Base
{
    public static function announce()
    {
        /** @var Admin\Helper\Nav $oNav */
        $oNav = Factory::factory('Nav', 'nails/module-admin')
            ->setLabel('Books')
            ->addAction('All Books', 'index');
            ->addAction('Reviews', 'reviews');

        return $oNav;
    }

    public function index()
    {
        Admin\Helper::loadView('index');
    }

    public function reviews()
    {
        Admin\Helper::loadView('reviews');
    }
}
```

Note how the `loadView` method does not specify the directory, it is inferred from the controller's name.

{% hint style="warning" %}
File casing is important! The view's directory must match the controller's casing _exactly_.
{% endhint %}

### View Data

Pass data to the view by assigning values to the controller's `$data` property:

```php
public function reviews()
{
    $oModel = Factory::model('Reviews', 'app');
    
    $this->data['aReviews'] = $oModel->getAll();
    
    Admin\Helper::loadView('reviews');
}
```

And the corresponding view file, `Book/reviews.php`:

```php
<table>
    <tbody>
        <?php
        foreach ($aReviews as $oReview) {
            ?>
            <tr>
                <td><?=$oReview-body?></td>
            </tr>
            <?php
        }
        ?>
    </tbody>
</table>
```
