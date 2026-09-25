---
description: Add widgets to the admin dashboard, and raise alerts at the top of it.
---

# Dashboard Widgets

The admin dashboard is a grid that each user arranges for themselves. Users add widgets from a picker, then drag, resize, configure and remove them. Admin saves each user's layout.

Any component can supply widgets. Admin discovers them the same way it discovers controllers.

## Creating a widget

A widget is a class in the component's `Admin\Dashboard\Widget` namespace that implements `Nails\Admin\Interfaces\Dashboard\Widget`. For your app, that's `src/Admin/Dashboard/Widget/`.

The console can create one for you:

```bash
nails make:admin:dashboard:widget
```

The `Nails\Admin\Traits\Dashboard\Widget` trait provides sensible defaults, so you only need to write the title, description and body:

```php
<?php
// src/Admin/Dashboard/Widget/RecentBooks.php

namespace App\Admin\Dashboard\Widget;

use Nails\Admin\Interfaces;
use Nails\Admin\Traits;
use Nails\Factory;

class RecentBooks implements Interfaces\Dashboard\Widget
{
    use Traits\Dashboard\Widget;

    public function getTitle(): string
    {
        return 'Recent Books';
    }

    public function getDescription(): string
    {
        return 'The most recently added books.';
    }

    public function getBody(): string
    {
        $aBooks = Factory::model('Book', 'app')->getAll([
            'sort'  => [['created', 'desc']],
            'limit' => 5,
        ]);

        $sOut = '<ul>';
        foreach ($aBooks as $oBook) {
            $sOut .= '<li>' . htmlspecialchars($oBook->label) . '</li>';
        }
        $sOut .= '</ul>';

        return $sOut;
    }
}
```

The widget now appears in the dashboard's "add widget" picker.

### The interface

| Method                                     | Trait default | Purpose                                                                                           |
| ------------------------------------------ | ------------- | ------------------------------------------------------------------------------------------------- |
| `__construct(array $aConfig = [])`         | Stores `$this->aConfig` | Receives the user's saved configuration for this instance.                              |
| `getTitle(): string`                       | —             | Shown in the widget header and the picker.                                                        |
| `getDescription(): string`                 | —             | Shown in the picker.                                                                              |
| `getImage(): ?string`                      | `null`        | An optional preview image URL for the picker.                                                     |
| `getBody(): string`                        | —             | The widget's HTML.                                                                                |
| `isEnabled(?User $oUser = null): bool`     | `true`        | Return `false` to hide the widget, for example when the user lacks a [permission](user-permissions.md). |
| `isPadded(): bool`                         | `true`        | Whether the body gets standard padding. Return `false` for edge-to-edge content such as charts.   |
| `isConfigurable(): bool`                   | `false`       | Whether to show the configure button.                                                             |
| `getConfig(): string`                      | `''`          | HTML for the configuration form.                                                                  |

{% hint style="info" %}
If a widget throws while it renders, Admin leaves it off the dashboard rather than failing the page. If a widget goes missing, check your logs.
{% endhint %}

### Restricting a widget

```php
use App\Admin\Permission;
use Nails\Auth\Resource\User;

public function isEnabled(?User $oUser = null): bool
{
    return userHasPermission(Permission\Book\Browse::class, $oUser);
}
```

### Configurable widgets

A configurable widget returns form fields from `getConfig()`. When the user saves the form, each field's `name` and `value` is saved for that instance of the widget. The values are passed back to the constructor as `$aConfig` whenever the widget is rendered.

```php
public function isConfigurable(): bool
{
    return true;
}

public function getConfig(): string
{
    return form_field_dropdown([
        'key'     => 'limit',
        'label'   => 'Books to show',
        'default' => $this->aConfig['limit'] ?? 5,
        'options' => [5 => 5, 10 => 10, 20 => 20],
    ]);
}

public function getBody(): string
{
    $iLimit = (int) ($this->aConfig['limit'] ?? 5);
    // ...
}
```

Each instance has its own config, so a user can add the same widget twice with different settings.

## Dashboard alerts

Alerts are banners at the top of the dashboard that flag something needing attention, such as a failed job or a missing setting. They aren't per-user and can't be dismissed for good. An alert shows whenever its condition is true.

An alert is a class in the component's `Admin\Dashboard\Alert` namespace (`src/Admin/Dashboard/Alert/` in your app) that implements `Nails\Admin\Interfaces\Dashboard\Alert`:

```php
<?php
// src/Admin/Dashboard/Alert/MissingIsbns.php

namespace App\Admin\Dashboard\Alert;

use Nails\Admin\Interfaces\Dashboard\Alert;
use Nails\Factory;

class MissingIsbns implements Alert
{
    protected int $iCount;

    public function __construct()
    {
        $this->iCount = Factory::model('Book', 'app')->countAll([
            'where' => [['isbn', null]],
        ]);
    }

    public function getTitle(): ?string
    {
        return 'Books missing an ISBN';
    }

    public function getBody(): ?string
    {
        return sprintf('%d books have no ISBN.', $this->iCount);
    }

    public function getSeverity(): string
    {
        return static::SEVERITY_WARNING;
    }

    public function isAlerting(): bool
    {
        return $this->iCount > 0;
    }
}
```

`getSeverity()` returns one of `SEVERITY_SUCCESS`, `SEVERITY_INFO`, `SEVERITY_WARNING` or `SEVERITY_DANGER`.

Admin creates every alert on each dashboard load, so keep the constructor and `isAlerting()` cheap.
