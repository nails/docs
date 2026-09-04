---
description: This helper provides an API for generating <picture> elements
---

# Picture

`<picture>` elements provide a means for the browser to contextually load images appropriate to the user's device. It's a bad idea to server an image designed for a 2K monitor to a user on a portrait mobile device.

{% embed url="https://developer.mozilla.org/en-US/docs/Web/HTML/Element/picture" %}

Use the `\Nails\Cdn\Helper\Picture` class to generate `<picture>` markup in an object orientated way:

```php
use Nails\Cdn\Helper\Picture;

// This value can be an object's ID or an instance of \Nails\Cdn\Resource\CdnObject
$mCdnObject = 123;

// Define the size of the fallback image, the alt text you'd like to
// use (optional), as well as any attributes (as a key/value array)
$iWidth      = 1600;
$iHeight     = 900;
$sAlt        = 'Cras mattis consectetur purus sit amet fermentum.';
$aAttributes = [
    'loading' => 'lazy',
];

$oPicture = new Picture($mCdnObject, $iWidth, $iHeight, $sAlt, $aAttributes);

// Add sources for each breakpoint or pixel density you'd like to support
// This method accepts: width, height, breakpoint, density
$picture
    ->source(800, 800, 768)
    ->source(400, 200, 340)
    ->source(800, 800, null, 1.5);
    
// Generate the markup (casting $picture as a string will also work)
echo $picture->generate();
```
