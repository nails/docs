---
description: Configuring Nails is simple using Environment variables and global constants.
---

# Configuration

Configuration in Nails uses a relatively simple cascading priority system which checks environment variables, defined constants, and default values and stores them as a flat key/value pair array.

## Getting Config Values

Retrieve config values using the Config class' `get` method:

```php
\Nails\Config::get('CONFIG_NAME', 'Default Value');
```

This will return the value of the `CONFIG_NAME` config in the following order:

1. Any [previously set value](configuration.md#setting-config-values)
2. A constant called `CONFIG_NAME`
3. An env var called `CONFIG_NAME`
4. The default value passed as the second parameter

{% hint style="success" %}
Strings which are valid JSON will be automatically decoded 🤯
{% endhint %}

## Setting Config Values

Typically values are set using environment variables or constants. However if you need to set something explicitly you can do so using the `config:set()` method, which accepts two arguments:

1. The name of the config to set
2. The value to set

```php
\Nails\Config::set('CONFIG_NAME', 'The value to set');
```

{% hint style="info" %}
If you are using the [Docker Environment](http://docker.nailsapp.co.uk), set your configurations in `docker-compose.yml` and/or `docker-compose.override.yml`
{% endhint %}

## Avoiding Environment Variables

If you cannot, or will not, use Environment Variables you can specify values by defining constants in one of two config files: `config/app.php` and `config/deploy.php`. Each file behaves the same way but has a slightly different semantic purpose. These files will be loaded early in execution and can be used to define configuration values.

### `config/app.php`

This file should contain any configurations which do not vary between deployments, for example, the app's name, timezone, etc.  For example:

```php
define('APP_NAME', 'My Nails App');
define('THIRDPARTY_ID', '1234');
```

{% hint style="info" %}
It is expected that if you are using file-based configuration that you will commit this file to version control.
{% endhint %}

### `config/deploy.php`

This file should contain any configurations which _will_ vary between deployments, for example, database credentials, private keys, third party tokens. For example:

```php
define('THIRD_PARTY_SECRET', 'abc-1234');
```

{% hint style="info" %}
It is expected that this file will **not** be committed to version control.
{% endhint %}

{% hint style="success" %}
When choosing which file a particular constant should go in consider whether it needs to change between your local, staging, or production environments. If it's consistent throughout (e.g the app's name) then it belongs in `config/app.php`; if it might change between staging and production (e.g. the Base URL) then it belongs in `config/deploy.php`.

Another consideration is whether a value might change between _machines_ in a multi-server environment. If this is the case then `config/deploy.php` should be used as your deployment process can place the appropriate configs per machine.
{% endhint %}
