---
description: >-
  Opt-in breadcrumb trails for admin pages, with optional links for each crumb.
---

# Breadcrumbs

Admin pages can opt in to a formal breadcrumb trail, shown at the top of the view (for example `Admin › Books › Create`). Each crumb has a label and may optionally be a link.

Pages that never push crumbs keep the existing title-based header (`setTitles()` or the legacy `$this->data['page']->title` string). When a trail is present, it is used for the visible header and the document `<title>` is kept in sync from the crumb labels.

## Adding crumbs

Call `addBreadcrumb()` from any controller that extends `Nails\Admin\Controller\Base`. The first call automatically prepends **Admin**, linked to the dashboard.

```php
public function reviews(): void
{
    $this
        ->addBreadcrumb('Books', static::url())
        ->addBreadcrumb('Reviews', static::url('reviews'));

    Helper::loadView('reviews');
}
```

The example above renders:

```
Admin › Books › Reviews
```

`Admin` links to the dashboard, `Books` to the controller index, and `Reviews` to the `reviews` method. Omit the URL (or pass `null`) if a crumb should not be a link:

```php
$this->addBreadcrumb('Preview');
```

`prependBreadcrumb()` inserts a crumb immediately after **Admin**, which is useful when a nested action needs an intermediate step added after the trail has already been seeded.

## The `DefaultController`

`DefaultController` opts in for you. The index is the tip of the trail:

| Action | Trail |
|--------|--------|
| index | `Admin › {plural}` |
| create | `Admin › {plural} › Create` |
| edit | `Admin › {plural} › Edit` |
| sort | `Admin › {plural} › Sort` |

`{plural}` comes from `CONFIG_TITLE_PLURAL` (falling back to the model name) and links to the controller index. Override `getIndexBreadcrumbLabel()` if you need a different label.

Nested actions can push extra crumbs after `setBreadcrumbTrail()`:

```php
public function import(): void
{
    $this
        ->setBreadcrumbTrail('Create', static::url('create'))
        ->addBreadcrumb('Import', static::url('import'))
        ->loadView('import');
}
```

That produces `Admin › Books › Create › Import`.

If you override `index()`, `create()`, `edit()`, or `sort()` without calling `setBreadcrumbTrail()` (or `parent::{method}()`), the page stays on the title-based header.

## Factory and service

Crumbs are `Nails\Admin\Factory\Breadcrumb` instances held by the request-scoped `Nails\Admin\Service\Breadcrumb` trail. Controllers normally go through `addBreadcrumb()`, but you can work with them directly:

```php
use Nails\Admin\Constants;
use Nails\Admin\Factory\Breadcrumb;
use Nails\Admin\Service\Breadcrumb as BreadcrumbService;
use Nails\Factory;

/** @var Breadcrumb $oCrumb */
$oCrumb = Factory::factory('Breadcrumb', Constants::MODULE_SLUG);
$oCrumb
    ->setLabel('Reviews')
    ->setUrl(static::url('reviews'));

/** @var BreadcrumbService $oTrail */
$oTrail = Factory::service('Breadcrumb', Constants::MODULE_SLUG);
$oTrail
    ->add('Books', static::url())
    ->add($oCrumb);
```

The service also supports `prepend()`, `reset()`, `getItems()`, `getLabels()`, and `isEmpty()`.
