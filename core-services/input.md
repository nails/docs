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

## Request encoding

Before any controller, model, or database query runs, Nails checks that every key and value in `$_GET`, `$_POST`, and `$_COOKIE` is valid UTF-8 (using `mb_check_encoding()`). If any of them aren't, the request is rejected with a `400 Bad Request` and a short plain text body; nothing is stripped or sanitised. This applies to both HTML and API requests. CLI requests are never rejected.

Newlines, tabs, and other valid UTF-8 control characters are unaffected; this check only rejects malformed byte sequences, such as overlong-encoded lead bytes.
