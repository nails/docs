---
description: How components add Settings pages, where values are stored, and who can open them.
---

# Settings pages

Any installed component can add a page under the _Settings_ sidebar group. Admin discovers classes in the component's `Settings` namespace that implement `Nails\Common\Interfaces\Component\Settings`, and renders one page per class.

You do not write a controller or a view. The shared Settings controller in `nails/module-admin` builds the form from the fields you return.

## Where settings classes live

| Component | Namespace | Location |
| --- | --- | --- |
| App | `App\Settings` | `src/Settings/` |
| Module | `{Module}\Settings` | `src/Settings/` |
| Driver | `{DriverNamespace}\Settings` | `src/{Driver}/Settings/` |

## Writing a settings class

The class must implement three methods:

```php
<?php
// src/Settings/Shop.php

namespace App\Settings;

use App\Admin\Permission;
use Nails\Common\Interfaces;
use Nails\Components\Setting;
use Nails\Factory;

class Shop implements Interfaces\Component\Settings
{
    const KEY_CURRENCY = 'currency';

    public function getLabel(): string
    {
        return 'Shop';
    }

    public function getPermission(): string
    {
        return Permission\Settings\Shop::class;
    }

    public function get(): array
    {
        /** @var Setting $currency */
        $currency = Factory::factory('ComponentSetting');
        $currency
            ->setKey(static::KEY_CURRENCY)
            ->setLabel('Default currency')
            ->setFieldset('General');

        return [$currency];
    }
}
```

* `getLabel()` is the sidebar link and the page title.
* `getPermission()` is the class-string of the permission that guards the page. It is required.
* `get()` returns `Nails\Components\Setting` objects. Group them with `setFieldset()`; each fieldset becomes a tab.

The permission class is a normal [admin permission](user-permissions.md), discovered in `Admin\Permission`. Put it next to the settings class it guards:

```php
<?php
// src/Admin/Permission/Settings/Shop.php

namespace App\Admin\Permission\Settings;

use Nails\Admin\Interfaces\Permission;

class Shop implements Permission
{
    public function label(): string
    {
        return 'Can manage Shop settings';
    }

    public function group(): string
    {
        return 'Settings';
    }
}
```

{% hint style="warning" %}
`getPermission()` replaced `getPermissions()`. A settings class that does not implement `getPermission(): string` will fatal on load. A leftover `getPermissions()` is ignored.
{% endhint %}

## Who can open a page

A user can open and save a settings page when any of these is true:

1. They are a **super user**.
2. They hold `Nails\Admin\Admin\Permission\Settings\Manage` ("Can manage all application settings"). This is sudo for settings: it unlocks every page.
3. They hold **that page's own permission**, returned by `getPermission()`.

Otherwise the page is hidden from the sidebar, and visiting its URL is unauthorised.

Grant the page permission when you want a group to manage one settings screen. Grant `Manage` when they should manage every settings screen.

## Where values are stored

Saved values go in `app_setting`. The grouping is the component's Composer slug (`app`, `nails/module-auth`, `nails/driver-cdn-local`, and so on). The key is the field's `setKey()` value.

Read them with `appSetting()`:

```php
use App\Settings\Shop;

$currency = appSetting(Shop::KEY_CURRENCY, 'app');
```

For a module, pass that module's `Constants::MODULE_SLUG`.

Fields marked `setIsReadOnly(true)` are shown but not saved. Encrypted fields use `setEncrypted(true)`.

## Drivers and skins

Driver and skin settings appear in the same sidebar group, with a badge naming the module they belong to. They are guarded the same way: a permission class under the driver's `Admin\Permission\Settings` namespace, returned from `getPermission()`.
