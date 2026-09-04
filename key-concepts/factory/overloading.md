---
description: >-
  Easily extend components by overloading their services, models, resources, and
  factories to change how they behave.
---

# Overloading

In some situations the functionality provided by an installed component isn't quite enough. You may wish to add additional methods to a [resource](resources.md), change the behaviour of a [service](services.md), or change the default values of a [factory](factories.md).

This is easily achieved through a combination of class inheritance and the [Factory definitions](./#defining-factory-items).

{% hint style="info" %}
There are two ways to overload [Factory](./) functionality, the way you choose depends on the item being overloaded and how it has been implemented.
{% endhint %}

## Provide an "app version" of a class

More often than not, components will check for the existence of an "app version" of the class being loaded, and if present instantiate that instead of the bundled version. It is expected (and often enforced) that the app version of a class be a child of the bundled version.

If we look art, for example, the [nails/common services.php](https://github.com/nails/common/blob/master/services/services.php) file, you will see that most items have a `class_exists` check:

```php
use Nails\Common\Service;

return [
    'services' => [
        'Asset' => function (): Service\Asset {
            if (class_exists('\App\Common\Service\Asset')) {
                return new \App\Common\Service\Asset();
            } else {
                return new Service\Asset();
            }
        },
    ]
];
```

Here, we are checking for the presence of `\App\Common\Service\Asset` and if it's there, return a new instance of it. Additionally, we are enforcing that the instance be a child of the parent via the closure's return type hint.

In order to extend `\Nails\Common\Service\Asset` you simply create `\App\Common\Service\Asset` and make sure it extends the parent. The Factory will then give you an instance of your class whenever `Factory::service('Asset')` is called.

{% hint style="success" %}
Unsure what to name your class, or if it can be extended at all? Then check out the component in question's `application/services.php` file and determine the logic being used; this directory is always at the root folder of the component.
{% endhint %}

## Overload the services.php file

The brute-force way of changing the item which is returned by the factory is to overload the component's `services.php` file from within the app's `application/services` directory.

For example, say we discovered a bug which only occurred at noon on a leap year, we could make debugging easier by overloading the `DateTime` factory provided by `nails/common` to always return a `\DateTime` object set at `2020-02-29 12:00:00`. All code utilising it would then get the modified date/time rather than the current date/time (which is the default behaviour).

To achieve this we start by creating an override `services.php` file in the app's `application/services/` directory which is in a similar directory structure as the component we're overloading:

```
./application/services/nails/common/services.php
```

This file will be detected by the [Factory](./) and will be loaded _instead_ of the file defined by `nails/common`.

Within this file you must return a new services array with any changed you wish to make. It is therefore important that you `require` the parent `services.php` file.

```php
# Load the parent services array at the beginning of the file
$aServices = require 'vendor/common/services/services.php';

# Overwrite the key you wish to change
$aServices['factories']['DateTime'] = function() {
    return new \DateTime('2020-02-29 12:00:00');
}

# Return the updated services array at the end of the file
return $aServices;
```

With the above in place all calls to `Factory::factory('DateTime')` will return the modified instance of `\DateTime` – much easier than changing the system clock!

{% hint style="warning" %}
This approach only alters the instance of the object returned by the [Factory](./) - in this example calling `new \DateTime()` would still give you a "now" `\DateTime` object.
{% endhint %}

