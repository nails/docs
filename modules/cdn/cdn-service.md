---
description: >-
  The CDN Service provides a comprehensive suite of methods for interacting with
  the configured CDN Driver.
---

# CDN Service

Load the CDN Service using the Factory:

```php
use Nails\Cdn;

/** @var Cdn\Service\Cdn $oCdn */
$oCdn = Factory::service('Cdn', Cdn\Constants::MODULE_SLUG);
```
