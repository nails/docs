---
description: >-
  Next we'll understand how the assets system works in Nails and learn how to
  style our pages.
---

# Javascript and CSS

Nails expects compiled Javascript and CSS assets to live in the `./www/assets/build/` directory in `js` and `css` sub-directories respectively.

How those compiled files are generated is not a concern of Nails, however the app ships with a default [Webpack](javascript-and-css.md#webpack) config which you are welcome to use.

## The Assets service

The [Asset service](../core-services/asset.md) is a convenience service which allows you to load both compiled assets, as well as inject inline javascript and CSS to any page.

If you were paying attention on the previous step you might have noticed that in the app's `Base` controller we are using the [Asset service](../core-services/asset.md) to load the app's global javascript file and stylesheet:

```php
/** @var \Nails\Common\Service\Asset $oAsset **/
$oAsset = \Nails\Factory::service('Asset');
$oAsset
    ->load('app.css')
    ->load('app.js');
```

The service's `load()` method here is automatically detecting the file type based on the file extension, and will load the file from the application's `./www/assets/build/*/` directory (`js` or `css` depending on the file type).&#x20;

## Webpack

Shipping by default with Nails is a basic Webpack config which will compile SASS and JS from the application's `./www/assets/*/` directory.

Running `make build` will trigger the assets to be compiled, additionally `make watch` will start a watch process, compiling assets on-the-fly as changes are saved.
