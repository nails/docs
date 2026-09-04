---
description: >-
  The SEO module brings functionality for improving site SEO and social
  presence.
---

# SEO

## Installation

Install this module using composer:

```bash
compsoer require nails/module-seo
```

## Configuration

There are no configuration options available for this module.

## Robots.txt

If you wish, you can generate your application's `robots.txt` file dynamically using this module. You might wish to do this if you want to offer different behaviour depending on context, e.g. which environment is running (block all non-production environments).

This is achieved by creating classes in the `App\Seo\Robots\UserAgent` namespace which implement the `Nails\Seo\Interfaces\Robots\UserAgent` interface.

{% hint style="info" %}
The `Nails\Seo\Factory\Robots\UserAgent` class can be extended which offers sensible defaults.
{% endhint %}

The following example uses the helper class, targets all user agents, and disallows crawling of the entire site, but only on `STAGING` and `DEVELOPMENT` environments:

```php
namespace App\Seo\Robots\UserAgent;

use Nails\Environment;
use Nails\Seo\Factory\Robots\UserAgent;

class Star extends UserAgent
{
    public function userAgent(): string
    {
        return '*';
    }

    public static function appliesTo(): array
    {
        return [
            Environment::ENV_STAGE,
            Environment::ENV_DEV,
        ];
    }

    public function disallow(): array
    {
        return array_merge(
            parent::disallow(),
            [
                '/',
            ]
        );
    }
}
```

If you wished to provide a separate set of directives to `GoogleBot` you might create the following in addition to the above:

```php
namespace App\Seo\Robots\UserAgent;

use Nails\Environment;
use Nails\Seo\Factory\Robots\UserAgent;

class GoogleBot extends UserAgent
{
    public function userAgent(): string
    {
        return 'GoogleBot';
    }

    public function priority(): string
    {
        // Use the priority value to order useragents
        return parent::priority() + 1;
    }

    public static function appliesTo(): array
    {
        return [
            Environment::ENV_STAGE,
            Environment::ENV_DEV,
        ];
    }

    public function disallow(): array
    {
        return [
            '/hide-from-google',
        ];
    }
}
```

Here we are placing the `GoogleBot` entry after the `Star` entry (using the `priority` method), and giving it a different set of `disallow` URLs; it also applies to `STAGING` and `DEVELOPMENT` environments only.
