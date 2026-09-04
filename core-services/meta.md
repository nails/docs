---
description: The Meta service is responsible for managing the application's <meta> tags.
---

# Meta

The Meta service is loaded using the [Factory](../key-concepts/factory/):

```php
use Nails\Common\Service\Meta;
use Nails\Factory;

/** @var Meta $oMeta */
$oMeta = Factory::service('Meta');
```

## Setting page meta data (Open Graph etc)

Common page meta data (SEO meta, OpenGraph, and Twitter card tags) is set using the `$this->oMetaData` property of controllers. This, by default, is populated using any knowledge which can be inferred by the request - this is typically minimal but will include the site's name (via the `APP_NAME` [configuration](../getting-started/configuration.md)) and URL.

The easiest way to set default meta data values (e.g. description and image)  is to extend the `MetaData` class and set the default values while constructing, for example:

```php
namespace App\Common\Resource;

class MetaData extends \Nails\Common\Resource\MetaData
{
    public function __construct($mObj = [])
    {
        parent::__construct($mObj);
        $this
            ->setDescription('The default description for your site.')
            ->setImageUrl(siteUrl('assets/img/default-share-image.jpg'))
            ->setImageWidth(1200)
            ->setImageHeight(1200);
    }
}
```

This will set the values _site-wide_, you can then override these values on a page-by-page basis. For example in the context of a single article page, you could set the page title to the article's title, and the image to the article's featured image:

```php
$this->oMetaData
    ->setTitles([
        $oArticle->label
    ])
    ->setImageUrl(
        cdnServe($oArticle->featured_image_id)
    );
```

{% hint style="info" %}
The `setTitles()` method accepts an array of titles which will be joined using the `MetaData` class' `$sTitleSeparator` property - useful for when you need to show hierarchy in the page title.
{% endhint %}
