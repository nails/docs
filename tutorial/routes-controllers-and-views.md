---
description: >-
  Next we briefly understand routes and how modules, controllers, and views work
  together.
---

# Routes and Modules

Nails loosely follows an [HMVC architecture](https://en.wikipedia.org/wiki/Hierarchical_model%E2%80%93view%E2%80%93controller). Controllers and Views are grouped into modules, which are closely coupled to the URL structure.

## Routes

Routing is Nails can be both automatic, explicit, or a combination of the two. For the purposes of this tutorial we are going to leverage the automatic routes - where the URL maps to specific controllers in the application's directory structure. It is highly recommended that you read the [Routing section](../key-concepts/routing.md) to understand how routes are resolved.

{% content-ref url="../key-concepts/routing.md" %}
[routing.md](../key-concepts/routing.md)
{% endcontent-ref %}

## Modules

Modules are the building blocks of any Nails application. They can be summarised as being classes which represent controllers, and PHP files which are views grouped together in a directory at `./www/application/modules` directory.

See the following page for more information on modules:

{% content-ref url="../key-concepts/components/modules.md" %}
[modules.md](../key-concepts/components/modules.md)
{% endcontent-ref %}

Already in place is the `home` module, which is mapped to the site's [default root](../key-concepts/routing.md#default-route-home-page). If we inspect this we can understand what is happening.

### The Home controller

Open `./www/application/modules/home/controllers/Home.php` in your IDE:

```php
use Nails\Factory;
use App\Controller\Base;

class Home extends Base
{
    public function index(): void
    {
        Factory::service('View')
            ->load([
                'structure/header/blank',
                'home/index',
                'structure/footer/blank',
            ]);
    }
}
```

Here we see the controller's default method `index()` rendering three views:

1. `structure/header/blank`
2. `home/index`
3. `structure/footer/blank`

Views 1 and 3 are default views, provided by `nails/common` which provide standard header and footer HTML, essentially leaving you to fill in the blanks between the `<body>` tags.

View 2 is the main body of the page and can be found in `./www/application/modules/home/views` and is where the body of the homepage is marked up.

### The Base controller

You will notice that the `Home` controller extends `Nails\Common\Controller\Base`. The `Base` controller's responsibility is to set up the application for every request. In the context of a typical website, this will be to do common tasks such as loading global Javascript and CSS, or loading the site's primary navigation menu from the database.

Before we get our hands dirty and write our own modules, let's understand how Nails handles static assets (Javascript and CSS) and how we can style our pages.
