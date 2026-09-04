---
description: The View service is responsible for rendering templates.
---

# View

The View service is loaded using the [Factory](../key-concepts/factory/):

```php
use Nails\Common\Service\View;
use Nails\Factory;

/** @var View $oView */
$oView = Factory::service('View');
```

