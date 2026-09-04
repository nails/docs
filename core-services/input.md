---
description: >-
  The Input service is responsible for data which comes as part of the request,
  e.g. $_POST, $_GET, $_SERVER, etc.
---

# Input

The Input service is loaded using the [Factory](../key-concepts/factory/):

```php
use Nails\Common\Service\Input;
use Nails\Factory;

/** @var Input $oInput */
$oInput = Factory::service('Input');
```

The following methods are available, and are fairly self-explanatory:

```php
$oInput->post($sKey);
$oInput->get($sKey);
$oInput->server($sKey);
```

{% hint style="info" %}
If no `$sKey` is provided, then the entire array is returned.
{% endhint %}
