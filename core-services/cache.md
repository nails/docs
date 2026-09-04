---
description: >-
  The FileCache service provides a simple interface for reading and writing from
  the application's cache directory.
---

# FileCache

The FileCache service provides an API for writing to, and reading from a dedicated cache directory on disk.

The FileCache service is loaded using the [Factory](../key-concepts/factory/):

```php
use Nails\Common\Service\FileCache;
use Nails\Factory;

/** @var FileCache $oFileCache */
$oFileCache = Factory::service('FileCache');
```

## Private Cache

The private cache is a cache which is designed to be private to the application, which no direct access via a URL.&#x20;

```php
// Returns the cache directory
$oFileCache->getDir();

$sCacheKey = 'my-cahced-item';
$sData     = 'Some data to write';

// Write a new item to the cache
$oFileCache->write($sData, $sCacheKey);

// Read an existing item from the cache
$oFileCache->read($sCacheKey);

// Determine whether a cache key exists
$oFileCache->exists($sCacheKey);
```

Items returned from the cache are instances of `Nails\Common\Resource\FileCache\Item` - these objects wrap the item on disk but do not access the file until cast as a string, allowing you to efficiently read cache items without having to hit the disk until actually required.

## Public Cache

The public cache has an identical interface to the private cache, however it is exposed to the public via a URL and adds the `getUrl(string $sKey = null): string` method.

Access the public cache via the FileCache service's `public()` method:

```php
$oPublicCache = $oFileCache->public();

$sCacheKey = 'my-cahced-item';
$sData     = 'Some data to write';

// Write a new item to the cache
$oPublicCache->write($sData, $sCacheKey);

// Read an existing item from the cache
$oPublicCache->read($sCacheKey);

// Determine whether a cache key exists
$oPublicCache->exists($sCacheKey);

// Get the item's URL
$oPublicCache->getUrl($sCacheKey);
```

{% hint style="info" %}
If you need to change the URL for the public cache, perhaps to route via a CDN, then set the `CACHE_PUBLIC_URL` [configuration option](../getting-started/configuration.md).
{% endhint %}

## Configuration

The following configuration options can be set to customise behaviour:

| Option              | Description                                 | Default           |
| ------------------- | ------------------------------------------- | ----------------- |
| `CACHE_PRIVATE_DIR` | The directory to use for the private cache. | `./cache/private` |
| `CACHE_PUBLIC_DIR`  | The directory to use for the public cache.  | `./cache/public`  |
| `CACHE_PUBLIC_URL`  | The base URL for items in the public cache. | `/cache/public`   |

