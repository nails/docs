---
description: How Admin decides who can see and do what, and how to define your own permissions.
---

# User Permissions

Admin access is set per [user group](../auth.md). Each group has an access control list (ACL): the permissions its members have. The rules are:

* A user is an **admin** if their group's ACL has at least one permission. Anyone else is turned away at the `/admin` router.
* A user is a **super user** if their group has the `Nails\Admin\Admin\Permission\SuperUser` permission. Super users pass every permission check.
* Everyone else can do what their group's permissions allow.

Before any of that, Admin can also restrict access by IP address. See [Restricting admin by IP](#restricting-admin-by-ip).

You grant permissions when editing a user group in admin. The list there is built from every permission class discovered in the installed components, grouped by component.

## Defining a permission

A permission is a small class that implements `Nails\Admin\Interfaces\Permission`. Admin discovers it in the component's `Admin\Permission` namespace, which for your app is `src/Admin/Permission/`:

```php
<?php
// src/Admin/Permission/Book/Edit.php

namespace App\Admin\Permission\Book;

use Nails\Admin\Interfaces\Permission;

class Edit implements Permission
{
    public function label(): string
    {
        return 'Can edit books';
    }

    public function group(): string
    {
        return 'Books';
    }
}
```

* `label()` is the text shown next to the checkbox when editing a group.
* `group()` is the heading it's listed under. Permissions from one component that share a group are shown together.

The permission's identity is its fully qualified class name. That's what is stored in the group's ACL and what you pass when checking it. Sub-namespaces are just for organisation. A common layout is one directory per area and one class per action:

```
src/Admin/Permission/
  Book/
    Browse.php
    Create.php
    Edit.php
    Delete.php
  Report/
    View.php
```

## Checking a permission

Pass the class name (or an instance) to `userHasPermission()`:

```php
use App\Admin\Permission;

if (!userHasPermission(Permission\Book\Edit::class)) {
    unauthorised();
}
```

Passing anything that isn't a permission class throws a `Nails\Admin\Exception\PermissionException`. An empty value always passes.

Check permissions in both places they matter:

* **In `announce()`**, so the sidebar only shows links the user can use.
* **In the action itself**, because users can type URLs.

[`DefaultController`](controllers/default-controller.md#permissions) does both for you. Set its `CONFIG_PERMISSION_*` constants.

### Helper functions

These global functions are always loaded:

| Function                                                          | Returns `true` when                                                                                                        |
| ----------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------- |
| `userHasPermission($mPermission, ?User $oUser = null)`            | The user (default: the active user) has the permission, or is a super user.                                                |
| `userHasAnyPermission(array $aPermissions, ?User $oUser = null)`  | The user has at least one of the permissions.                                                                              |
| `groupHasPermission($mPermission, ?Group $oGroup = null, bool $bIgnoreSuperUser = false)` | The group has the permission. Pass `true` as the third argument to ignore super-user status. |
| `isSuperUser(?User $oUser = null)`                                | The user's group has the `SuperUser` permission.                                                                           |
| `isGroupSuperUser(?Group $oGroup = null)`                         | The group has the `SuperUser` permission.                                                                                  |
| `isAdmin(?User $oUser = null)`                                    | The user's group has any admin permission.                                                                                 |

They wrap the `Permission` service (`Factory::service('Permission', 'nails/module-admin')`), which also has `get()` and `getGrouped()` for listing every discovered permission.

## Renaming or moving permissions

ACLs store class names, so renaming or moving a permission class removes it from every group that had it. To carry grants across, write a [migration](../../core-services/database/migrations.md) that uses the `Nails\Admin\Traits\Database\Migration\PermissionMap` trait, and set a `MAP` of old names to new ones:

```php
<?php

namespace App\Database\Migration;

use Nails\Admin\Traits\Database\Migration\PermissionMap;
use Nails\Common\Interfaces;
use Nails\Common\Traits;

class Migration5 implements Interfaces\Database\Migration
{
    use Traits\Database\Migration;
    use PermissionMap;

    const MAP = [
        'App\Admin\Permission\Book\Manage' => 'App\Admin\Permission\Book\Edit',
        'App\Admin\Permission\Book\Legacy' => '',
    ];
}
```

The trait supplies the migration's `execute()` method, which rewrites each user group's ACL. Names that aren't in the map are left alone. Mapping a name to an empty string removes that permission from every group.

## Restricting admin by IP

_Settings → Admin → IP Whitelist_ limits admin to a list of IP addresses. Enter one entry per line, or separate entries with commas. Each entry is either an exact address (`203.0.113.7`) or an IPv4 CIDR range (`203.0.113.0/24`). IPv6 ranges aren't supported, so list IPv6 addresses individually. The form shows your current IP address, so you can include it.

When the list isn't empty, a request from any other address:

* gets a **404** from `/admin`, even for a logged-in super user, so admin looks as if it doesn't exist
* fails authentication on admin API endpoints: those that extend `Nails\Admin\Controller\BaseApi` or use the `Nails\Admin\Traits\Api\RestrictToAdmin` trait

An empty list means no IP restriction.

To check an address in your own code, use the `Permission` service:

```php
use Nails\Admin\Constants;
use Nails\Admin\Service\Permission;
use Nails\Factory;

/** @var Permission $oPermission */
$oPermission = Factory::service('Permission', Constants::MODULE_SLUG);

$oPermission->isIpAllowed();              // The current request's IP
$oPermission->isIpAllowed('203.0.113.7'); // A specific IP
$oPermission->getIpWhitelist();           // The configured entries
```

{% hint style="warning" %}
If you save a list that doesn't include your own address, you'll be locked out of admin straight away. To recover, clear the `whitelist` app setting in the `nails/module-admin` group directly in the database, or from a machine whose IP is on the list.
{% endhint %}

The whitelist only covers admin. Front-end routes and other modules' APIs are unaffected. If you need to restrict a whole site, do it at the web server or proxy.
