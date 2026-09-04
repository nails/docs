---
description: Easily scale and crop images stored in the CDN.
---

# Image Transformation

## Cropping

## Scaling

## Security

Whilst image manipulation via the URL is incredibly useful it is quite dangerous to leave this open. An abuser could quite quickly overwhelm the server by generating many variations of an image.

All components, including the application, must explicitly announce the image dimensions they wish the CDN to honour; this is done in each component's `composer.json` file:

```javascript
{
    "extra": {
        "nails": {
            "data": {
                "nails/module-cdn": {
                    "permitted-image-dimensions": [
                        "120x120",
                        "250x250"
                    ]
                }
            }
        }
    }
}
```

The array will accept values which match the following:

* `/^\dx\d$/i`
* `/^\d$/`
* `[$iWidth, $iHeight]`

If a value cannot be parsed then a `\Nails\Cdn\Exception\PermittedDimensionException` exception will be thrown. This exception will also be thrown if you attempt to generate, or visit, a URL which results in an invalid dimension being requested.

{% hint style="warning" %}
When in `PRODUCTION` an exception will not be thrown when visiting an invalid URL, instead the user will receive a 404.
{% endhint %}

If you wish to disable this protection, set the [Configuration](../../getting-started/configuration.md) value `CDN_ALLOW_DANGEROUS_IMAGE_TRANSFORMATION` to `true`.
