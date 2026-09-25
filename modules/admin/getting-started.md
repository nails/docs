---
description: Build a working admin screen for a model, then lock it down with permissions.
---

# Your First Admin Page

This walkthrough adds a _Books_ section to admin for a `Book` model provided by your app. It uses the three pieces you'll use most: a controller, permissions, and (optionally) a custom view.

It assumes you already have a `Book` [model](../../key-concepts/factory/models/) and [resource](../../key-concepts/factory/resources.md) registered with the app, and that you can log in to `/admin`.

{% stepper %}
{% step %}
### Create the controller

Admin controllers go in `src/Admin/Controller/`, in the `App\Admin\Controller` namespace. Extend `DefaultController` and tell it which model to manage:

```php
<?php
// src/Admin/Controller/Book.php

namespace App\Admin\Controller;

use Nails\Admin\Controller\DefaultController;

class Book extends DefaultController
{
    const CONFIG_MODEL_NAME     = 'Book';
    const CONFIG_MODEL_PROVIDER = 'app';
    const CONFIG_SIDEBAR_ICON   = 'fa-book';
}
```

Reload admin. A **Books** group appears in the sidebar with a **Manage Books** link, and you can browse, search, create, edit, delete and restore books. `DefaultController` builds the forms from the model's field definitions, so there are no views to write.

The screen is served from `/admin/app/book`. The URL is made from the component (`app`) and the class name (`Book` becomes `book`). See [Routing](controllers/#routing).
{% endstep %}

{% step %}
### Tune the index

Most of what `DefaultController` does is controlled by constants. For example, to choose the index columns, the sort options and some fields to skip:

```php
const CONFIG_INDEX_FIELDS = [
    'Title'     => 'label',
    'Author'    => 'author',
    'Published' => 'is_published',
    'Modified'  => 'modified',
];

const CONFIG_SORT_OPTIONS = [
    'Title'    => 'label',
    'Modified' => 'modified',
];

const CONFIG_EDIT_READONLY_FIELDS = ['isbn'];
```

[Default Controller](controllers/default-controller.md) lists every option, plus the hooks (`beforeEdit()`, `afterCreate()` and so on) for adding behaviour.
{% endstep %}

{% step %}
### Add permissions

Right now every admin user can manage books. To restrict it, define a permission class for each action you want to control. Permissions go in `src/Admin/Permission/`:

```php
<?php
// src/Admin/Permission/Book/Browse.php

namespace App\Admin\Permission\Book;

use Nails\Admin\Interfaces\Permission;

class Browse implements Permission
{
    public function label(): string
    {
        return 'Can browse books';
    }

    public function group(): string
    {
        return 'Books';
    }
}
```

Create `Create`, `Edit` and `Delete` in the same way, then point the controller at them:

```php
use App\Admin\Permission;

const CONFIG_PERMISSION_BROWSE = Permission\Book\Browse::class;
const CONFIG_PERMISSION_CREATE = Permission\Book\Create::class;
const CONFIG_PERMISSION_EDIT   = Permission\Book\Edit::class;
const CONFIG_PERMISSION_DELETE = Permission\Book\Delete::class;
```

The new permissions show up, under a heading for your app, when you edit a user group. Grant them to the groups that need them. Super users always have every permission. See [User Permissions](user-permissions.md).
{% endstep %}

{% step %}
### Add a custom page

When you need a screen that isn't CRUD, add a method and a view. Here's a `stats` page on the same controller:

```php
use Nails\Admin\Factory\Nav;
use Nails\Admin\Helper;

public static function announce(): Nav|array|null
{
    // Keep the "Manage Books" link DefaultController adds, and add another
    $oNav = parent::announce();

    if (userHasPermission(Permission\Book\Browse::class)) {
        $oNav->addAction('Book Stats', 'stats');
    }

    return $oNav;
}

public function stats(): void
{
    if (!userHasPermission(Permission\Book\Browse::class)) {
        unauthorised();
    }

    $this
        ->setBreadcrumbTrail('Stats')
        ->setData('iTotal', static::getModel()->countAll());

    Helper::loadView('stats');
}
```

Put the view in `application/modules/admin/views/Book/stats.php`. The directory name matches the controller's class name exactly, including case:

```php
<div class="group-stats">
    <p>There are <strong><?=number_format($iTotal)?></strong> books.</p>
</div>
```

It's served at `/admin/app/book/stats`, with the breadcrumb trail `Admin › Books › Stats`.
{% endstep %}
{% endstepper %}

## Next steps

* [Controllers](controllers/) explains discovery, routing, view resolution, and how to override a module's controller.
* [Forms](forms.md) and [Helper](helper/) cover the building blocks for custom views.
* [Dashboard Widgets](dashboard-widgets.md) and [Data Export](data-export.md) are the other common extension points.
