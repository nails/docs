---
description: >-
  The Asset service is responsible for loading JS, CSS - both external, and
  inline.
---

# Asset

The Asset service provides a single API for loading and unloading Javascript and CSS assets at runtime.

The Asset service is loaded using the [Factory](../key-concepts/factory/):

```php
use Nails\Common\Service\Asset;
use Nails\Factory;

/** @var Asset $oAsset */
$oAsset = Factory::service('Asset');
```

## Loading assets

@todo - write this

## Static Assets

The above loading method is designed for loading JS and CSS. However there are other items you may wish to load from the assets directory:

* Images
* Fonts
* Media

You can use the `asset(string $sAsset): string` helper to load these files throughout your application (e.g. in views or resources).

This will return a URL for the asset, taking into consideration any [custom URLs](asset.md#changing-the-url) or [cache busting](asset.md#cache-busting) which might be configured.

```php
<img src="<?=asset('img/logo.png')?>">
```

## Critical CSS

Critical CSS is the act of inlining CSS which applies to the "above the fold" elements of the page and deferring the load of the otherwise blocking main stylesheet. When the stylesheet is loaded it then takes over. The result is a page which seemingly loads much faster than it actually does.

The Asset service offers a mechanic for setting the deferred stylesheet and any inline CSS you wish to load via it's `criticalCss()` method. This returns an instance of `\Nails\Common\FactoryAsset\CriticalCss`.

```php
/** @var \Nails\Common\Service\Asset $oAsset */
$oAsset = Factory::service('Asset');
$oAsset
    ->criticalCss()
    // This will be loaded in the same way as $oAsset->load() 
    ->setDeferredStylesheet('app.css')
    ->setInlineCss([
        // You can pass CSS file paths
        '/path/to/some.css',
        // Or normal CSS
        '.class {margin: none;}',
    ]);
```

{% hint style="info" %}
Do not load your main style sheet as well as use Critical CSS — it'll be loaded twice.
{% endhint %}

The above will result in something like the following at the top of the page's output:

```markup
<!DOCTYPE html>
<html lang="en">
<head>
    <title>My Site</title>
    
    <!-- The inline CSS -->
    <style type="text/css">
    /* The contents of /path/to/some.css */
    .class {margin: none;}
    </style>
    
    <!-- The deferred stylesheet -->
    <link rel="stylesheet"
          as="style"
          href="https://localhost/assets/build/css/app.css"
          media="print"
          onload="this.media='all'"
    />
    
    ...
```

{% hint style="info" %}
Note that if there is no inline CSS to render then the deferred stylesheet will be loaded as normal, i.e without the `onload` attribute etc.
{% endhint %}

## Cache busting

A speedy website makes good use of caching, both browser caching and maybe even CDNs. This can introduce issues when the original file is changed but the URL does not.

Nails solves this by allowing you to generate a "cache buster" – a string which is appended to the end of all assets forcing the browser, or CDN, to re-cache the item.

Configure this by defining the `ASSET_REVISION` [configuration option](../getting-started/configuration.md); set this to a string, and change it when needed (for example, when deploying).

{% hint style="success" %}
A good cache buster to use is the currently deployed git commit hash.
{% endhint %}

## Changing the URL

By default, assets are loaded from the app's `/assets` directory; compiled assets (such as JS and CSS)  are served form the `build` subdirectory (in `js` and `css` subdirectories, respectively). For example, the app's JS and CSS might be loaded like this:

```php
use Nails\Common\Service\Asset;
use Nails\Factory;

/** @var Asset $oAsset */
$oAsset = Factory::service('Asset');
$oAsset
    ->load('app.min.js')
    ->load('app.min.css');
```

Or, an image rendered on the page might be be loaded like this:

```php
<img src="<?asset('img/logo.png')?>">
```

With the following URLs being generated:

```
https://mysite.com/assets/build/js/app.min.js
https://mysite.com/assets/build/css/app.min.css
https://mysite.com/assets/img/logo.png
```

It is possible to customise the URL from which assets are served. This is useful, for example, if you wish to place a CDN over this directory to increase performance.

Imagine we created a new CDN and configured its origin as `https://mysite.com/assets` and set it up so it was accessed via `https://static.mysite.com`.&#x20;

We can [configure](../getting-started/configuration.md) the Asset service to use our new URL by setting the following configuration value:

| Option      | Value                       |
| ----------- | --------------------------- |
| `ASSET_URL` | `https://static.mysite.com` |

Now the output URLs of the above example assets would be:

```
https://static.mysite.com/build/js/app.min.js
https://static.mysite.com/build/css/app.min.css
https://static.mysite.com/img/logo.png
```

Our CDN would cache these requests at various edges and subsequent requests would be much faster.

{% hint style="info" %}
This will also perform [cache busting](asset.md#cache-busting) if configured.
{% endhint %}

## Additional Configuration Options

The following configuration values can be adjusted for this service:

| Option             | Description                               | Default     |
| ------------------ | ----------------------------------------- | ----------- |
| `ASSET_REVISION`   | The cache busting string to use.          | `null`      |
| `ASSET_URL`        | The URL to use for serving static assets. | `/assets`   |
| `ASSET_URL_SECURE` | The secure version of the above           | `/assets`   |
| `ASSET_CSS_DIR`    | The directory for compiled CSS assets     | `build/css` |
| `ASSET_JS_DIR`     | The directory for compiled JS assets.     | `build/js`  |

