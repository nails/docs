---
description: >-
  The Error Handler service is responsible for processing and delegating all
  errors which occur within the application
---

# Error Handler

All errors in Nails are funnelled to the ErrorHandler service. This service is responsible for reporting errors to the user, as well as logging. Responsibility for _how_ errors are handled is delegated to the enabled [handler](./#handlers).

The Error Handler service is loaded using the [Factory](../../key-concepts/factory/):

```php
use Nails\Common\Service\ErrorHandler;
use Nails\Factory;

/** @var ErrorHandler $oErrorHandler */
$oErrorHandler = Factory::service('ErrorHandler');
```

## Handlers

### Default Handler

The default handler is reserved on production environments, and verbose everywhere else.&#x20;

{% content-ref url="default-handler.md" %}
[default-handler.md](default-handler.md)
{% endcontent-ref %}

### Rollbar Handler

Forwards issues to [Rollbar](https://rollbar.com), then delegates to the [Default Handler](./#default-handler).

{% content-ref url="rollbar-handler.md" %}
[rollbar-handler.md](rollbar-handler.md)
{% endcontent-ref %}

## Templates

### HTML / CLI

There are two sets of error templates: HTML and CLI. The HTML templates are intended to be seen by front-end users, and the CLI are for tasks which happen on the command line.

### Overriding Templates

All templates can be overridden by the application by providing a specific template in the app's `application/views/error` directory.

{% tabs %}
{% tab title="404" %}
This template is displayed when a 404 is encountered.

* HTML: `application/views/errors/html/404.php`
* CLI: `application/views/errors/cli/404.php`
{% endtab %}

{% tab title="401" %}
This template is displayed when a 401 is encountered.

* HTML: `application/views/errors/html/401.php`
* CLI: `application/views/errors/cli/401.php`
{% endtab %}

{% tab title="500" %}
This template is displayed when a 500 is encountered.

* HTML: `application/views/errors/html/500.php`
* CLI: `application/views/errors/cli/500.php`
{% endtab %}

{% tab title="Maintenance" %}
This template is displayed when the site is in maintenance mode.

* HTML: `application/views/errors/html/maintenance.php`
* CLI: `application/views/errors/cli/maintenance.php`

{% hint style="warning" %}
Note: this view is not displayed using the [View service](../view.md), but rather as a simple `require` very early on in execution; therefore you must provide an entire view (i.e header/footer) and not rely on anything which is made available during controller construction.
{% endhint %}
{% endtab %}
{% endtabs %}

{% hint style="info" %}
Remember, it is your responsibility to render any appropriate variables (e.g. error messages), and include any surrounding views (e.g. header/footer).
{% endhint %}
