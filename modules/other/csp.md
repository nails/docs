---
description: Build a Content-Security-Policy in code and send it as a response header.
---

# CSP

This module lets you build a [Content-Security-Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CSP) in PHP and sends it as a response header. You load a policy into the `Csp` service during the request. Just before output, the module compiles the policy and sets the `Content-Security-Policy` header. If no policy is loaded, no header is sent.

```bash
composer require nails/module-csp
```

## Loading a policy

Build a policy with the service's `make()` method, then `load()` it. Every `make()` argument is optional and is named after the directive in camel case (`script-src` → `scriptSrc`), so pass only the directives you need as named arguments:

```php
use Nails\Csp\Constants;
use Nails\Factory;

/** @var \Nails\Csp\Service\Csp $csp */
$csp = Factory::service('Csp', Constants::MODULE_SLUG);

$csp->load(
    $csp->make(
        defaultSrc: ['self'],
        scriptSrc: ['self', 'www.googletagmanager.com'],
        styleSrc: ['self', 'fonts.googleapis.com'],
        fontSrc: ['self', 'fonts.gstatic.com'],
        imgSrc: ['*', 'data:'],
        objectSrc: ['none'],
    )
);
```

That sends:

```
Content-Security-Policy: default-src 'self'; font-src 'self' fonts.gstatic.com; img-src * data:; object-src 'none'; script-src 'self' www.googletagmanager.com; style-src 'self' fonts.googleapis.com;
```

Load the policy somewhere that runs on every request you want covered, such as the constructor of your app's base controller. The header is set on the `Nails\Common\Events::OUTPUT_PRE` event, so the policy can change at any point before output.

| Method | Purpose |
|---|---|
| `make(...)` | Returns a new `Nails\Csp\Factory\Csp` policy. It does not load it. |
| `load($policy)` | Makes the policy the one that will be sent. Replaces any policy already loaded. |
| `get()` | Returns the loaded policy, or `null`. |
| `unload()` | Removes the loaded policy, so no header is sent. Use this for responses that must not carry the app's policy. |

### Supported directives

`base-uri`, `block-all-mixed-content`, `child-src`, `connect-src`, `default-src`, `font-src`, `form-action`, `frame-ancestors`, `frame-src`, `img-src`, `manifest-src`, `media-src`, `navigate-to`, `object-src`, `prefetch-src`, `report-uri`, `sandbox`, `script-src`, `script-src-attr`, `style-src`, `style-src-attr`, `style-src-elem`, `upgrade-insecure-requests`, and `worker-src`.

Directives are always written in that order. A directive with no values is left out.

### Keywords and sources

Write keywords without quotes. The module adds the quotes for these keywords:

`none`, `self`, `strict-dynamic`, `unsafe-eval`, `unsafe-inline`, `wasm-unsafe-eval`

Every other value is written exactly as you pass it, so schemes (`data:`, `blob:`, `wss://example.com`), hosts (`*.example.com`), and `*` need no special handling. To use a keyword the module does not quote, such as `'unsafe-hashes'` or a hash source, include the quotes yourself.

## Changing a policy

`make()` returns a `Nails\Csp\Factory\Csp` object with a getter and setter for every directive (`getScriptSrc()`, `setScriptSrc()`, and so on). Setters are chainable:

* A string appends one value to the directive.
* An array replaces every value in the directive.

```php
$csp
    ->get()
    ->setConnectSrc('api.example.com')      // add one source
    ->setFrameAncestors(['none']);          // replace the directive
```

Because the loaded policy is an object, a controller can add the sources one page needs without rebuilding the whole policy.

Call `compile()`, or cast the policy to a string, to get the header value without sending it.

## Nonces

A nonce lets a specific inline `<script>` or `<style>` run without allowing every inline script. Call `generateNonce()` on the policy to create a random nonce for the request, or `setNonce()` to use your own:

```php
$csp->load(
    $csp
        ->make(
            defaultSrc: ['self'],
            scriptSrc: ['self', 'strict-dynamic'],
            styleSrc: ['self'],
        )
        ->generateNonce()
);

$nonce = $csp->get()->getNonce();
```

When a policy has a nonce, the module adds `'nonce-{nonce}'` to `script-src`, `script-src-attr`, `style-src`, `style-src-attr`, and `style-src-elem`.

{% hint style="warning" %}
The nonce is added to all five directives, including any you did not set. A directive you set takes precedence over `default-src`, so setting a nonce changes the effective policy for those five. Set each of them explicitly when you use a nonce.
{% endhint %}

A directive that contains `unsafe-inline` does not get the nonce. Browsers ignore `unsafe-inline` when a nonce is present, so adding one would block inline code you meant to allow.

Put the nonce on each inline tag that should run:

```php
<script nonce="<?=$nonce?>">
    // …
</script>
```

To have the [Asset service](../../core-services/asset.md) add the nonce to the tags it outputs, pass the nonce to it:

```php
/** @var \Nails\Common\Service\Asset $asset */
$asset = Factory::service('Asset');
$asset->setNonce($nonce);
```

## Overriding the service and factory

The module's service and factory can be replaced by app classes. If the class exists, the module builds it instead of its own:

| Name | App class |
|---|---|
| `Csp` service | `App\Csp\Service\Csp` |
| `Csp` factory | `App\Csp\Factory\Csp` |

Each app class should extend the module class it replaces. An app service is a good place to keep named policies:

```php
namespace App\Csp\Service;

class Csp extends \Nails\Csp\Service\Csp
{
    public function frontEnd(): self
    {
        return $this->load(
            $this
                ->make(
                    defaultSrc: ['self'],
                    scriptSrc: ['self', 'strict-dynamic'],
                    styleSrc: ['self', 'unsafe-inline'],
                    objectSrc: ['none'],
                )
                ->generateNonce()
        );
    }
}
```

The app's base controller can then call `$csp->frontEnd()`.
