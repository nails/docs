---
description: All admin controllers must extend the base controller.
---

# Base Controller

All admin controllers must extend `Nails\Admin\Controller\Base`. Admin controllers route in a similar way to application controllers, but also usually implement to additional methods: `announce` and `permissions`.

### Announcing Controllers

The controller's `annoucne` method explicitly announces the controller's presence and returns an array of `actions` which the controller would like to register in admin's sidebar:

```php
public static function announce()
{
    return Factory::factory('Nav', 'nails/module-admin')    
        ->setLabel('Books')
        ->setIcon('fa-book')
        ->addAction('Manage Books', 'index')
        ->addAction('Manage Reviews', 'reviews');
}
```

This announcement specifies that the `Manage Books` action (which points to this controller's `index` method) should be registered to the `Books` sidebar group; it also specifies a second action for managing reviews.

{% hint style="info" %}
This method can return an array of `Nav` objects if you wish to add actions to multiple sidebar groups.
{% endhint %}

### Controller Permissions

By default, controllers are available to all admin users. If you wish to restrict your controller's functionality to specific user groups then you can return an array of permissions in the controller's `permissions` method:

```php
public static function permissions(): array
{
    return [
        'browse' => 'Can manage books',
        'edit'   => 'Can edit books',
        'delete' => 'Can delete books',
    ];
}
```

The above permissions are made available when managing user group permissions and you may test if a user has the permission using the `userHasPermission` function, passing in a string in the format `admin:app:{controller}:{permission}` as the argument.

```php
public function delete()
{
    if (!userHasPermission('admin:ap:books:delete')) {
        show404();
    }
    
    // ... deletion logic
}
```

See [User Permissions](../user-permissions.md) for a deeper dive into Admin's permission system.

