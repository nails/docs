---
description: All admin controllers extend the base controller.
---

# Base Controller

Every admin controller extends `Nails\Admin\Controller\Base`, either directly or through [`DefaultController`](default-controller.md). `Base` is abstract and implements `Nails\Admin\Interfaces\Controller`, so the one method you must write is `announce()`.

```php
<?php
// src/Admin/Controller/Report.php

namespace App\Admin\Controller;

use App\Admin\Permission;
use Nails\Admin\Constants;
use Nails\Admin\Controller\Base;
use Nails\Admin\Factory\Nav;
use Nails\Factory;

class Report extends Base
{
    public static function announce(): Nav|array|null
    {
        /** @var Nav $oNav */
        $oNav = Factory::factory('Nav', Constants::MODULE_SLUG);
        $oNav
            ->setLabel('Reports')
            ->setIcon('fa-chart-bar');

        if (userHasPermission(Permission\Report\View::class)) {
            $oNav->addAction('Sales Report', 'sales');
        }

        return $oNav;
    }

    public function sales(): void
    {
        if (!userHasPermission(Permission\Report\View::class)) {
            unauthorised();
        }

        $this
            ->addBreadcrumb('Reports')
            ->addBreadcrumb('Sales')
            ->setData('aRows', $this->getSalesData())
            ->loadView('sales');
    }
}
```

## What the constructor does

When Admin creates your controller, `Base::__construct()`:

1. Fires the `ADMIN:STARTUP` [event](../../../core-services/event.md).
2. Links `$this->data` to the main controller's data, so anything you set there reaches the view.
3. Loads the optional app configs `application/config/admin.php` and `application/modules/admin/config/admin.php`.
4. Clears any front-end assets, then loads Admin's own CSS and JS and the bundled libraries (jQuery, jQuery UI, Select2, CKEditor, Knockout, Moment, Mustache, Bootstrap, Font Awesome).
5. Loads anything components ask for in their `autoload` data (see below).
6. Fires the `ADMIN:READY` event.

If you override the constructor, call `parent::__construct()` first.

### Autoloading assets from a component

A component can ask Admin to load services, models, helpers, JS or CSS on every admin page by adding an `autoload` block for `nails/module-admin` to the `extra.nails.data` section of its `composer.json`:

```json
{
    "extra": {
        "nails": {
            "data": {
                "nails/module-admin": {
                    "autoload": {
                        "helpers": ["book"],
                        "assets": {
                            "js": ["admin.min.js"],
                            "css": ["admin.min.css"],
                            "jsInline": ["console.log('admin ready');"]
                        }
                    }
                }
            }
        }
    }
}
```

The supported keys are `services`, `models`, `helpers`, and `assets` with `js`, `jsInline`, `css` and `cssInline`.

## Announcing

`announce()` is static. Admin calls it on every discovered controller to build the sidebar, so keep it cheap. It returns a `Nav`, an array of `Nav` objects (to add links to several groups), or `null` (no sidebar presence). [Controllers](./#the-sidebar-announce) covers groups, icons, alerts and ordering.

## Permissions

Admin doesn't check permissions for you on a `Base` controller. Check them yourself, in two places:

* in `announce()`, so users only see links they can use
* in each method, because a user might visit a URL directly

Permissions are classes that implement `Nails\Admin\Interfaces\Permission`. Test them with `userHasPermission()`:

```php
if (!userHasPermission(Permission\Report\View::class)) {
    unauthorised();
}
```

See [User Permissions](../user-permissions.md) for how to define them.

## Methods

| Method                                        | Purpose                                                                                                          |
| --------------------------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| `static url(string $sUrl = ''): string`       | The absolute URL to this controller, with `$sUrl` appended. Follows [overrides](./#overriding-a-modules-controller). |
| `setTitles(array $aTitles)`                   | Sets the page header and document `<title>`. `Admin` is always added as the first segment.                       |
| `addBreadcrumb(string $sLabel, ?string $sUrl)` | Adds a crumb to a linked [breadcrumb trail](breadcrumbs.md). Replaces the title-based header.                   |
| `prependBreadcrumb(string $sLabel, ?string $sUrl)` | Inserts a crumb straight after the `Admin` root crumb.                                                      |
| `setData(string $sKey, mixed $mValue)`        | Makes a variable available to the view.                                                                          |
| `loadView(string $sView)`                     | A chainable shortcut for [`Helper::loadView()`](../helper/).                                                     |

Every method except `url()` returns `$this`, so calls can be chained.

### Page titles and breadcrumbs

`setTitles()` sets the header as a list of segments:

```php
$this->setTitles(['Books', 'Reviews']);   // Admin › Books › Reviews
```

To make individual segments into links, use `addBreadcrumb()` instead. See [Breadcrumbs](breadcrumbs.md).
