---
description: Serve a CMS page at the site root.
---

# Homepage

Two settings make a CMS page the site root.

## Set the default route

In `application/config/routes.php`, point the [default route](../../../key-concepts/routing.md#default-route-home-page) at the CMS homepage action:

```php
$route['default_controller'] = 'cms/render/homepage';
```

`cms/render/page` expects a page id and will 404 on its own. `homepage` loads the page chosen below and then renders it.

If that page's own slug is requested, the render controller redirects to `/` with a 301.

When no homepage is stored, rendering throws `Nails\Cms\Exception\RenderException\HomepageNotDefinedException`.

## Choose the page

In admin, open Settings → CMS and set **Homepage**. The dropdown lists published pages. The value is the page id, stored as the `homepage` app setting in the `nails/module-cms` grouping.
