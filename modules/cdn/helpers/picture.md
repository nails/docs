---
description: This helper provides an API for generating <picture> elements
---

# Picture

`<picture>` elements let the browser choose an image for the current viewport or pixel density. Serving one large crop to every device wastes bandwidth, and appending `@2x` to a URL is unreliable: once a crop is cached, `urlCrop()` returns the static cache file, and the `@2x` variant of that file does not exist.

{% embed url="https://developer.mozilla.org/en-US/docs/Web/HTML/Element/picture" %}

Use `\Nails\Cdn\Helper\Picture` to build the markup. Each size is a normal CDN crop, so a cached 1x file and its 2x counterpart are separate cache entries.

```php
use Nails\Cdn\Helper\Picture;

// An object ID, or an instance of \Nails\Cdn\Resource\CdnObject.
// Numeric strings and other objects are rejected.
$mCdnObject = 123;

// Fallback crop, shown when no <source> matches. Pass '' to omit alt.
// Attributes are written onto the <img>, not the <picture>.
$oPicture = new Picture($mCdnObject, 1600, 900, 'A mountain lake', null, [
    'loading' => 'lazy',
]);

// source() accepts width, height, breakpoint, density.
// The browser uses the first matching <source>, so add the most specific first.
$oPicture
    ->source(800, 800, 768)
    ->source(400, 200, 340)
    ->source(3200, 1800, null, 2);

// Casting to a string also calls generate().
echo $oPicture;
```

That renders, in source order:

```html
<picture>
    <source srcset="{crop 800x800}" media="(min-width: 768px)">
    <source srcset="{crop 400x200}" media="(min-width: 340px)">
    <source srcset="{crop 3200x1800} 2x" media="(min-resolution: 2dppx)">
    <img src="{crop 1600x900}" alt="A mountain lake" loading="lazy" />
</picture>
```

## `source()`

| Arguments | `media` | `srcset` |
| --- | --- | --- |
| Width, height, and a breakpoint | `(min-width: {breakpoint}px)` | The crop URL |
| Width, height, and a density, no breakpoint | `(min-resolution: {density}dppx)` | The crop URL plus a `{density}x` descriptor |
| Width, height, a breakpoint, and a density | `(min-width: {breakpoint}px)` only | The crop URL plus a `{density}x` descriptor |
| Width and height only | Attribute omitted, so the source matches everyone | The crop URL |

A breakpoint source matches every pixel density at that viewport. Because each source carries a single crop, a 1x screen that matches it still downloads that crop. To keep the fallback on 1x displays and serve a larger crop only on high-DPI screens, omit the breakpoint and pass the density:

```php
echo (new Picture($iObjectId, 35, 35, $sAlt))
    ->source(70, 70, null, 2);
```

```html
<picture>
    <source srcset="{crop 70x70} 2x" media="(min-resolution: 2dppx)">
    <img src="{crop 35x35}" alt="…" />
</picture>
```

If you add several density-only sources, put the highest density first. `(min-resolution: 2dppx)` also matches a 3x display, so a later 3x source would never be reached.

## Permitted dimensions

Every width and height, including the fallback, must be a [permitted image dimension](../image-transformation.md). `generate()` asks the CDN for each crop, and an unlisted size throws `\Nails\Cdn\Exception\PermittedDimensionException`.

## Constructor

```php
new Picture($mCdnObject, $iWidth, $iHeight, $sAlt, $oCdn, $aAttributes)
```

The fifth argument is an optional CDN service, used to inject a double in tests. HTML attributes are the sixth argument. Passing the attributes array as the fifth argument is a type error.
