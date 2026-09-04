---
description: This page explains how to configure a CMS page as the application's homepage.
---

# Homepage

It's straightforward to use a CMS page as the application's [default route](../../../key-concepts/routing.md#default-route-home-page). To achieve this there are two steps you need to take:

1. [Change the default route](homepage.md#setting-the-default-route)
2. [Define the default page in admin](homepage.md#setting-the-page-to-use-in-admin)

## Set the default route

The [default route](../../../key-concepts/routing.md#default-route-home-page) should be set to the CMS module's render controller, like so:

```php
$route['default_controller'] = 'cms/render/page';
```

## Set the page to use in admin

In admin, navigate to the CMS module's settings page where you can define the page you wish to use for the homepage.

{% hint style="info" %}
You can automate this part of the process using a [database migration](../../../core-services/database/migrations.md).
{% endhint %}
