---
description: The Session service is responsible for managing user sessions.
---

# Session

The Session service is loaded using the [Factory](../key-concepts/factory/):

```php
use Nails\Common\Service\Session;
use Nails\Factory;

/** @var Session $oSession */
$oSession = Factory::service('Session');
```

The session service uses native PHP Sessions, and is a thin wrapper of the [Symfony Session component](https://symfony.com/doc/current/components/http_foundation/sessions.html).

## Persistant Data

If you wish to save some data to the current session which persists until it is removed or the session expires. Then you have the following methods available to you:

### `setUserData(string $sKey, mixed $mValue): self`

Sets persistent data with key `$sKey` and value `$mValue`, returns an instance of the Session service for method chaining.

```php
$oSession->setUserData('key', 'value');
```

### `getUserData(string $sKey): mixed`

Returns any data saved with `setUserData()`.

```php
$mValue = $oSession->getUserData('key');
```

## Flash Data

Flash Data is data which is only available on the next page load - it is useful for status messages and hints after a redirect.&#x20;

### `setFlashData(string $sKey, mixed $mValue): self`

Sets flash data with key `$sKey` and value `$mValue`, returns an instance of the Session service for method chaining.

```php
$oSession->setFlashData('key', 'value');
```

### `getFlashData(string $sKey): mixed`

Returns any data saved with `setFlashData()`.

```php
$mValue = $oSession->getFlashData('key');
```

#### Reserved Flash Data

Nails has some reserved flash data keys, predominantly used for status feedback – feel free to read and write to them, however be aware that some processes might overwrite the value.

* `error` – Something went wrong
* `success` – Something went right
* `info` – You should know this
* `warning` – Something you may want to check happened

### `keepFlashData(string|array $mKey = null): self`

Will retain flash data for another page load. If keys are specified (as a string or an array) just those specific keys will be kept, if null is passed, or no keys are specified then all keys are kept

```php
// Keep all keys
$oSession->keepFlashData();

// Keep a single key
$oSession->keepFlashData('key');

// Keep multiple keys
$oSession->keepFlashData(['key1', 'key2']);
```

This is useful in redirect chains, for example if the homepage is only available to logged out users and logged in users are redirect to a dashboard - you might want to keep the login flash data so it is displayed on the dashboard:

```
Login – redirects to:
Home – keepFlashData(), redirects to:
Dashboard
```
